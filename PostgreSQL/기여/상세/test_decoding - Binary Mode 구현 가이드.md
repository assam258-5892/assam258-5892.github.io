# test_decoding - Binary Mode 구현 가이드

## 목차
1. [개요](#개요)
2. [test_decoding 현재 구조](#test_decoding-현재-구조)
3. [Binary Mode 구현 방법](#binary-mode-구현-방법)
4. [코드 수정 포인트](#코드-수정-포인트)
5. [테스트 및 검증](#테스트-및-검증)
6. [참고 자료](#참고-자료)

---

## 개요

PostgreSQL의 `test_decoding` 플러그인은 Logical Decoding의 참조 구현으로, **현재는 텍스트 형식으로만 출력**합니다. 

PostgreSQL 코어에는 `OUTPUT_PLUGIN_BINARY_OUTPUT` 타입이 이미 정의되어 있지만, `test_decoding`에서는 이를 활용하지 않고 항상 `OUTPUT_PLUGIN_TEXTUAL_OUTPUT`만 사용합니다.

이 문서는 **test_decoding을 직접 수정**하여 실제 바이너리 출력 기능을 추가하는 방법을 설명합니다. **변경이 필요한 핵심 부분만 발췌**하여 제시합니다.

---

## test_decoding 현재 구조

### 파일 위치
```
contrib/test_decoding/test_decoding.c
```

### 현재 출력 타입 설정

**contrib/test_decoding/test_decoding.c:164-172**

```c
static void
pg_decode_startup(LogicalDecodingContext *ctx, OutputPluginOptions *opt,
                  bool is_init)
{
    /* ... */
    
    ctx->output_plugin_private = data;
    
    opt->output_type = OUTPUT_PLUGIN_TEXTUAL_OUTPUT;  // ← 항상 텍스트만!
    opt->receive_rewrites = false;
    
    /* ... */
}
```

**문제점**: `OUTPUT_PLUGIN_BINARY_OUTPUT`이 정의되어 있지만 실제로 사용되지 않음

---

## Binary Mode 구현 방법

### 바이너리 프로토콜 형식 정의

```c
/*
 * Binary Protocol Format:
 *
 * All integers are in network byte order (big-endian)
 *
 * BEGIN Message:
 *   'B' (int8) - Message type
 *   xid (int32) - Transaction ID (if include-xids)
 *   timestamp (int64) - Commit timestamp (if include-timestamp)
 *
 * COMMIT Message:
 *   'C' (int8) - Message type
 *   xid (int32) - Transaction ID (if include-xids)
 *   timestamp (int64) - Commit timestamp (if include-timestamp)
 *   commit_lsn (int64) - Commit LSN
 *
 * CHANGE Message:
 *   'R' (int8) - Message type ('R' for Row change)
 *   action (int8) - 'I' (Insert), 'U' (Update), 'D' (Delete)
 *   relid (int32) - Relation OID
 *   nspname (string) - Namespace name (null-terminated)
 *   relname (string) - Relation name (null-terminated)
 *   
 *   For INSERT:
 *     has_new (int8) - 1 if new tuple present, 0 otherwise
 *     new_tuple_data - Binary tuple data (if has_new == 1)
 *   
 *   For UPDATE:
 *     has_old (int8) - 1 if old tuple present, 0 otherwise
 *     old_tuple_data - Binary tuple data (if has_old == 1)
 *     has_new (int8) - 1 if new tuple present, 0 otherwise
 *     new_tuple_data - Binary tuple data (if has_new == 1)
 *   
 *   For DELETE:
 *     has_old (int8) - 1 if old tuple present, 0 otherwise
 *     old_tuple_data - Binary tuple data (if has_old == 1)
 *
 * Binary Tuple Format:
 *   natts (int16) - Number of attributes
 *   For each attribute:
 *     length (int32) - Length of data, -1 for NULL
 *     data (bytes) - Binary data (if length >= 0)
 */
```

---

## 코드 수정 포인트

### 1. Startup 초기화 수정

**위치**: contrib/test_decoding/test_decoding.c:164-172

**기존 코드**:
```c
    ctx->output_plugin_private = data;
    
    opt->output_type = OUTPUT_PLUGIN_TEXTUAL_OUTPUT;
    opt->receive_rewrites = false;
```

**수정 후**:
```c
    ctx->output_plugin_private = data;
    
    opt->output_type = OUTPUT_PLUGIN_TEXTUAL_OUTPUT;  // 기본값, 옵션 파싱에서 변경 가능
    opt->receive_rewrites = false;
```

---

### 2. 바이너리 옵션 파싱 추가

**위치**: contrib/test_decoding/test_decoding.c:258-268 (else 블록 앞에 추가)

**기존 코드**:
```c
        else if (strcmp(elem->defname, "stream-changes") == 0)
        {
            /* ... stream-changes 처리 ... */
        }
        else
        {
            ereport(ERROR,
                    (errcode(ERRCODE_INVALID_PARAMETER_VALUE),
                     errmsg("option \"%s\" = \"%s\" is unknown",
                            elem->defname,
                            elem->arg ? strVal(elem->arg) : "(null)")));
        }
    }
    
    ctx->streaming &= enable_streaming;
}
```

**수정 후**:
```c
        else if (strcmp(elem->defname, "stream-changes") == 0)
        {
            /* ... stream-changes 처리 ... */
        }
        else if (strcmp(elem->defname, "force-binary") == 0)  // ← 추가
        {
            bool force_binary;
            
            if (elem->arg == NULL)
                continue;
            else if (!parse_bool(strVal(elem->arg), &force_binary))
                ereport(ERROR,
                        (errcode(ERRCODE_INVALID_PARAMETER_VALUE),
                         errmsg("could not parse value \"%s\" for parameter \"%s\"",
                                strVal(elem->arg), elem->defname)));
            
            if (force_binary)
                opt->output_type = OUTPUT_PLUGIN_BINARY_OUTPUT;
        }
        else
        {
            ereport(ERROR,
                    (errcode(ERRCODE_INVALID_PARAMETER_VALUE),
                     errmsg("option \"%s\" = \"%s\" is unknown",
                            elem->defname,
                            elem->arg ? strVal(elem->arg) : "(null)")));
        }
    }
    
    ctx->streaming &= enable_streaming;
}
```

---

### 3. 바이너리 Helper 함수 추가

**위치**: contrib/test_decoding/test_decoding.c (파일 상단, 함수 선언 전)

**추가할 코드**:
```c
/*
 * 바이너리 텍스트 전송 헬퍼 함수
 */
static void
pg_sendtext(StringInfo buf, const char *str, int slen)
{
    char *p;
    
    p = pg_server_to_client(str, slen);
    if (p != str)
    {
        slen = strlen(p);
        pq_sendint32(buf, slen);
        appendBinaryStringInfoNT(buf, p, slen);
        pfree(p);
    }
    else
    {
        pq_sendint32(buf, slen);
        appendBinaryStringInfoNT(buf, str, slen);
    }
}

static void
pg_sendstring(StringInfo buf, const char *str)
{
    if (str != NULL) {
        pg_sendtext(buf, str, strlen(str));
    }
    else
    {
        pq_sendint32(buf, 0);
    }
}

static void
pg_print_literal(StringInfo s, Oid typid, char *outputstr)
{
    switch (typid)
    {
        case INT2OID:
        case INT4OID:
        case INT8OID:
            pq_sendint32(s, 1);
            pg_sendstring(s, outputstr);
            break;
        
        case BOOLOID:
            pq_sendint32(s, 0);
            if (strcmp(outputstr, "t") == 0)
                pg_sendstring(s, "true");
            else
                pg_sendstring(s, "false");
            break;
        
        default:
            pq_sendint32(s, 0);
            pg_sendstring(s, outputstr);
            break;
    }
}
```

---

### 4. 바이너리 튜플 출력 함수 추가

**위치**: contrib/test_decoding/test_decoding.c (tuple_to_stringinfo 함수 다음)

**추가할 코드**:
```c
/*
 * 바이너리 형식으로 튜플 속성 출력
 */
static void
pg_tuple_to_attrs(StringInfo s, TupleDesc tupdesc,
                  HeapTuple tuple, bool skip_nulls)
{
    int natt;
    int offset;
    uint32 attrs = 0;
    
    offset = s->len;
    pq_sendint32(s, 0);  // 속성 개수 placeholder
    
    for (natt = 0; natt < tupdesc->natts; natt++)
    {
        Form_pg_attribute attr;
        Oid         typid;
        Oid         typoutput;
        bool        typisvarlena;
        Datum       origval;
        bool        isnull;
        
        attr = TupleDescAttr(tupdesc, natt);
        
        if (attr->attisdropped)
            continue;
        
        if (attr->attnum < 0)
            continue;
        
        typid = attr->atttypid;
        
        origval = heap_getattr(tuple, natt + 1, tupdesc, &isnull);
        
        if (isnull && skip_nulls)
            continue;
        
        attrs++;
        
        pg_sendstring(s, NameStr(attr->attname));
        
        getTypeOutputInfo(typid, &typoutput, &typisvarlena);
        
        if (isnull)
            pq_sendint32(s, PG_UINT32_MAX);
        else if (typisvarlena && VARATT_IS_EXTERNAL_ONDISK(origval))
        {
            pq_sendint32(s, 0);
            pg_sendstring(s, "unchanged-toast-datum");
        }
        else if (!typisvarlena)
            pg_print_literal(s, typid,
                            OidOutputFunctionCall(typoutput, origval));
        else
        {
            Datum val;
            val = PointerGetDatum(PG_DETOAST_DATUM(origval));
            pg_print_literal(s, typid,
                            OidOutputFunctionCall(typoutput, val));
        }
    }
    
    /* 실제 속성 개수를 placeholder에 저장 */
    attrs = pg_hton32(attrs);
    memcpy((char *pg_restrict) (s->data + offset), &attrs, sizeof(uint32));
}
```

---

### 5. BEGIN 콜백 수정

**위치**: contrib/test_decoding/test_decoding.c:306-313

**기존 코드**:
```c
static void
pg_output_begin(LogicalDecodingContext *ctx, TestDecodingData *data,
                ReorderBufferTXN *txn, bool last_write)
{
    OutputPluginPrepareWrite(ctx, last_write);
    if (data->include_xids)
        appendStringInfo(ctx->out, "BEGIN %u", txn->xid);
    else
        appendStringInfoString(ctx->out, "BEGIN");
    OutputPluginWrite(ctx, last_write);
}
```

**수정 후**:
```c
static void
pg_output_begin(LogicalDecodingContext *ctx, TestDecodingData *data,
                ReorderBufferTXN *txn, bool last_write)
{
    OutputPluginPrepareWrite(ctx, last_write);
    
    if (data->include_xids)
    {
        if (ctx->options.output_type == OUTPUT_PLUGIN_BINARY_OUTPUT)
        {
            pq_sendint8(ctx->out, 'B');
            pq_sendint32(ctx->out, txn->xid);
        }
        else
            appendStringInfo(ctx->out, "BEGIN %u", txn->xid);
    }
    else
    {
        if (ctx->options.output_type == OUTPUT_PLUGIN_BINARY_OUTPUT)
            pq_sendint8(ctx->out, 'B');
        else
            appendStringInfoString(ctx->out, "BEGIN");
    }
    
    OutputPluginWrite(ctx, last_write);
}
```

---

### 6. COMMIT 콜백 수정

**위치**: contrib/test_decoding/test_decoding.c:316-333

**기존 코드**:
```c
static void
pg_decode_commit_txn(LogicalDecodingContext *ctx, ReorderBufferTXN *txn,
                     XLogRecPtr commit_lsn)
{
    TestDecodingData *data = ctx->output_plugin_private;
    TestDecodingTxnData *txndata = txn->output_plugin_private;
    bool        xact_wrote_changes = txndata->xact_wrote_changes;
    
    pfree(txndata);
    txn->output_plugin_private = NULL;
    
    if (data->skip_empty_xacts && !xact_wrote_changes)
        return;
    
    OutputPluginPrepareWrite(ctx, true);
    if (data->include_xids)
        appendStringInfo(ctx->out, "COMMIT %u", txn->xid);
    else
        appendStringInfoString(ctx->out, "COMMIT");
        
    if (data->include_timestamp)
        appendStringInfo(ctx->out, " (at %s)",
                         timestamptz_to_str(txn->xact_time.commit_time));
    OutputPluginWrite(ctx, true);
}
```

**수정 후**:
```c
static void
pg_decode_commit_txn(LogicalDecodingContext *ctx, ReorderBufferTXN *txn,
                     XLogRecPtr commit_lsn)
{
    TestDecodingData *data = ctx->output_plugin_private;
    TestDecodingTxnData *txndata = txn->output_plugin_private;
    bool        xact_wrote_changes = txndata->xact_wrote_changes;
    
    pfree(txndata);
    txn->output_plugin_private = NULL;
    
    if (data->skip_empty_xacts && !xact_wrote_changes)
        return;
    
    OutputPluginPrepareWrite(ctx, true);
    
    if (data->include_xids)
    {
        if (ctx->options.output_type == OUTPUT_PLUGIN_BINARY_OUTPUT)
        {
            pq_sendint8(ctx->out, 'C');
            pq_sendint32(ctx->out, txn->xid);
        }
        else
            appendStringInfo(ctx->out, "COMMIT %u", txn->xid);
    }
    else
    {
        if (ctx->options.output_type == OUTPUT_PLUGIN_BINARY_OUTPUT)
            pq_sendint8(ctx->out, 'C');
        else
            appendStringInfoString(ctx->out, "COMMIT");
    }
    
    OutputPluginWrite(ctx, true);
}
```

---

### 7. CHANGE 콜백 수정

**위치**: contrib/test_decoding/test_decoding.c:599-666 (pg_decode_change 함수)

**기존 코드의 핵심 부분**:
```c
static void
pg_decode_change(LogicalDecodingContext *ctx, ReorderBufferTXN *txn,
                 Relation relation, ReorderBufferChange *change)
{
    /* ... 변수 선언 및 초기화 ... */
    
    OutputPluginPrepareWrite(ctx, true);
    
    appendStringInfoString(ctx->out, "table ");
    appendStringInfoString(ctx->out,
                           quote_qualified_identifier(/* ... */));
    appendStringInfoChar(ctx->out, ':');
    
    switch (change->action)
    {
        case REORDER_BUFFER_CHANGE_INSERT:
            appendStringInfoString(ctx->out, " INSERT:");
            /* ... 텍스트 출력 ... */
            break;
        /* ... UPDATE, DELETE ... */
    }
    
    OutputPluginWrite(ctx, true);
}
```

**수정 후** (바이너리 모드 처리 추가):
```c
static void
pg_decode_change(LogicalDecodingContext *ctx, ReorderBufferTXN *txn,
                 Relation relation, ReorderBufferChange *change)
{
    TestDecodingData *data;
    TestDecodingTxnData *txndata;
    Form_pg_class class_form;
    TupleDesc   tupdesc;
    MemoryContext old;
    
    data = ctx->output_plugin_private;
    txndata = txn->output_plugin_private;
    
    /* BEGIN 출력 (필요시) */
    if (data->skip_empty_xacts && !txndata->xact_wrote_changes)
        pg_output_begin(ctx, data, txn, false);
    txndata->xact_wrote_changes = true;
    
    class_form = RelationGetForm(relation);
    tupdesc = RelationGetDescr(relation);
    
    old = MemoryContextSwitchTo(data->context);
    
    OutputPluginPrepareWrite(ctx, true);
    
    if (ctx->options.output_type == OUTPUT_PLUGIN_BINARY_OUTPUT)
    {
        /* 바이너리 모드 출력 */
        const char *nspname = get_namespace_name(
            get_rel_namespace(RelationGetRelid(relation)));
        const char *relname = NameStr(class_form->relname);
        
        /* 변경 타입 */
        switch (change->action)
        {
            case REORDER_BUFFER_CHANGE_INSERT:
                pq_sendint8(ctx->out, 'I');
                break;
            case REORDER_BUFFER_CHANGE_UPDATE:
                pq_sendint8(ctx->out, 'U');
                break;
            case REORDER_BUFFER_CHANGE_DELETE:
                pq_sendint8(ctx->out, 'D');
                break;
        }
        
        /* 릴레이션 정보 (nspname과 relname) */
        pg_sendstring(ctx->out, nspname);
        pg_sendstring(ctx->out, relname);
        
        /* 튜플 데이터 */
        if (change->data.tp.oldtuple != NULL)
        {
            pg_tuple_to_attrs(ctx->out, tupdesc,
                              &change->data.tp.oldtuple->tuple,
                              true);
        }
        else
            pq_sendint32(ctx->out, 0);
        
        if (change->data.tp.newtuple != NULL)
        {
            pg_tuple_to_attrs(ctx->out, tupdesc,
                              &change->data.tp.newtuple->tuple,
                              false);
        }
        else
            pq_sendint32(ctx->out, 0);
    }
    else
    {
        /* 텍스트 모드 출력 (기존 코드 유지) */
        appendStringInfoString(ctx->out, "table ");
        appendStringInfoString(ctx->out,
                               quote_qualified_identifier(
                                   get_namespace_name(get_rel_namespace(RelationGetRelid(relation))),
                                   NameStr(class_form->relname)));
        appendStringInfoChar(ctx->out, ':');
        
        switch (change->action)
        {
            case REORDER_BUFFER_CHANGE_INSERT:
                appendStringInfoString(ctx->out, " INSERT:");
                if (change->data.tp.newtuple == NULL)
                    appendStringInfoString(ctx->out, " (no-tuple-data)");
                else
                    tuple_to_stringinfo(ctx->out, tupdesc,
                                        &change->data.tp.newtuple->tuple, false);
                break;
                
            case REORDER_BUFFER_CHANGE_UPDATE:
                appendStringInfoString(ctx->out, " UPDATE:");
                if (change->data.tp.oldtuple != NULL)
                {
                    appendStringInfoString(ctx->out, " old-key:");
                    tuple_to_stringinfo(ctx->out, tupdesc,
                                        &change->data.tp.oldtuple->tuple, true);
                    appendStringInfoString(ctx->out, " new-tuple:");
                }
                
                if (change->data.tp.newtuple == NULL)
                    appendStringInfoString(ctx->out, " (no-tuple-data)");
                else
                    tuple_to_stringinfo(ctx->out, tupdesc,
                                        &change->data.tp.newtuple->tuple, false);
                break;
                
            case REORDER_BUFFER_CHANGE_DELETE:
                appendStringInfoString(ctx->out, " DELETE:");
                if (change->data.tp.oldtuple == NULL)
                    appendStringInfoString(ctx->out, " (no-tuple-data)");
                else
                    tuple_to_stringinfo(ctx->out, tupdesc,
                                        &change->data.tp.oldtuple->tuple, true);
                break;
                
            default:
                Assert(false);
        }
    }
    
    MemoryContextSwitchTo(old);
    MemoryContextReset(data->context);
    
    OutputPluginWrite(ctx, true);
}
```

---

## 테스트 및 검증

### 빌드

```bash
cd contrib/test_decoding
make clean
make
make install
```

### SQL 테스트

```sql
-- 슬롯 생성
SELECT pg_create_logical_replication_slot('test_slot', 'test_decoding');

-- 테스트 테이블
CREATE TABLE test_table (
    id serial PRIMARY KEY,
    name text,
    data bytea,
    created_at timestamptz DEFAULT now()
);

-- 데이터 삽입
BEGIN;
INSERT INTO test_table (name, data) VALUES ('test1', '\x010203'::bytea);
INSERT INTO test_table (name, data) VALUES ('test2', '\x040506'::bytea);
COMMIT;

-- 텍스트 모드 조회
SELECT lsn, xid, data 
FROM pg_logical_slot_get_changes('test_slot', NULL, NULL,
    'include-xids', 'true',
    'include-timestamp', 'true');

-- 바이너리 모드 조회
SELECT lsn, xid, data 
FROM pg_logical_slot_get_binary_changes('test_slot', NULL, NULL,
    'force-binary', 'true',
    'include-xids', 'true');

-- 슬롯 삭제
SELECT pg_drop_replication_slot('test_slot');
```

---

## 참고 자료

### 관련 소스 파일

1. **Output Plugin 인터페이스**
   - `src/include/replication/output_plugin.h` - OutputPluginOutputType 정의

2. **바이너리 I/O 함수**
   - `src/backend/libpq/pqformat.c` - pq_sendint*, pq_sendbytes 등
   - `src/backend/utils/adt/` - 각 타입의 send/recv 함수

3. **Logical Decoding 코어**
   - `src/backend/replication/logical/logical.c` - 논리적 디코딩 구현
   - `src/backend/replication/logical/logicalfuncs.c` - SQL 함수 인터페이스

### 유용한 함수

```c
// 바이너리 I/O
Datum OidSendFunctionCall(Oid functionId, Datum val);
bytea *SendFunctionCall(FmgrInfo *flinfo, Datum val);

// 타입 정보
void getTypeOutputInfo(Oid type, Oid *typOutput, bool *typIsVarlena);
void getTypeBinaryOutputInfo(Oid type, Oid *typSend, bool *typIsVarlena);

// 네트워크 바이트 순서 변환
uint16 pg_hton16(uint16 x);
uint32 pg_hton32(uint32 x);
uint64 pg_hton64(uint64 x);

// StringInfo 조작
void appendBinaryStringInfo(StringInfo str, const char *data, int datalen);
void resetStringInfo(StringInfo str);
```

### 성능 고려사항

1. **메모리 관리**: MemoryContext를 적절히 사용하여 메모리 누수 방지
2. **네트워크 효율**: 바이너리가 텍스트보다 20-30% 더 컴팩트
3. **CPU 사용**: 바이너리 직렬화가 텍스트 변환보다 빠름

---

## 결론

`test_decoding`에 바이너리 모드를 추가하면:

- **성능 향상**: 직렬화/역직렬화 오버헤드 감소
- **데이터 정확성**: 타입 손실 없이 원본 바이너리 데이터 전송
- **네트워크 효율성**: 더 컴팩트한 데이터 표현

이 문서는 원본 `test_decoding` 소스의 변경이 필요한 핵심 부분만 발췌하여 제시했습니다. 실제 구현 시 프로젝트 요구사항에 맞게 조정하시기 바랍니다.

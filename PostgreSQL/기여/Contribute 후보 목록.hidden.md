# Contribute 후보 목록

## Table Access Method API 개선

## Page/Tuple 한계값 테스트 개선

## 에러 메시지 수정
다른 곳에는 다 포함된 "PostgreSQL"이 pg_backup에만 포함되지 않음.
```
src/bin/pg_basebackup/po/ko.po
#: pg_recvlogical.c:74
#, c-format
msgid ""
"%s controls PostgreSQL logical decoding streams.\n"
"\n"
msgstr ""
"%s 프로그램은 논리 디코딩 스트림을 제어하는 도구입니다.\n"
"\n"
"%s 프로그램은 PostgreSQL 논리 디코딩 스트림을 제어하는 도구입니다.\n"
"\n"
```

번역이 변경되며 아래의 문장이 사라짐.
```
src/bin/psql/po/zh-TW.po
<<<<<<< HEAD
"***(按 Enter 鍵繼續或輸入 x 然後按 Enter 鍵取消)********************\n"
||||||| parent of d11fa348335 (TDE 번역: eXperDB-TDE 브랜딩)
"***(按 Enter 鍵繼續或鍵入 x 來取消)********************\n"

# help.c:83
#: help.c:82
#, c-format
msgid ""
"psql is the PostgreSQL interactive terminal.\n"
"\n"
msgstr ""
"psql 是 PostgreSQL 文字模式介面。\n"
>>>>>>> d11fa348335 (TDE 번역: eXperDB-TDE 브랜딩)
"\n"
```

타밀어 번역이 음차로 되어 있음.
```
#: pg_config.c:398
#, c-format
msgid ""
"\n"
"%s provides information about the installed version of PostgreSQL.\n"
"\n"
msgstr ""
"\n"
" %s நிறுவப் பட்டுள்ள போஸ்ட்கிரெஸ் வெள்யீடு குறித்தத் தகவலைத் தருகிறது.\n"
"\n"

" %s நிறுவப் பட்டுள்ள PostgreSQL வெள்யீடு குறித்தத் தகவலைத் தருகிறது.\n"
```
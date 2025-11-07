# CloudBerry Contributor 가이드

CloudBerry Database는 Apache 인큐베이터 프로젝트로, 분산형 PostgreSQL 기반 데이터베이스입니다. 이 가이드는 CloudBerry 프로젝트에 기여하기 위한 상세한 절차를 제공합니다.

## 1. CloudBerry Database 이해

### 1.1 프로젝트 개요
- **Apache Incubator 프로젝트**: 2023년부터 Apache Software Foundation에서 인큐베이팅
- **분산 아키텍처**: Coordinator-Segment 구조의 MPP(Massively Parallel Processing) 데이터베이스
- **PostgreSQL 기반**: PostgreSQL 호환성 제공하면서 분산 처리 능력 확장
- **오픈소스**: Apache License 2.0 하에 배포

### 1.2 기술적 특징
- **분산 쿼리 처리**: 여러 노드에서 병렬 처리
- **컬럼형 저장**: 분석 워크로드 최적화
- **벡터화 실행**: SIMD 명령어를 활용한 고성능 처리
- **확장성**: 수평적 스케일링 지원

### 1.3 아키텍처 이해
- **Coordinator Node**: 쿼리 계획 및 조정 역할
- **Segment Nodes**: 실제 데이터 저장 및 처리
- **Interconnect**: 노드 간 통신 레이어
- **Resource Management**: 리소스 할당 및 관리

## 2. 개발 환경 설정

### 2.1 시스템 요구사항
- **운영체제**: Linux (CentOS 7+, Ubuntu 18.04+, Rocky Linux 8+)
- **메모리**: 최소 8GB, 권장 16GB 이상
- **디스크**: 최소 50GB 여유 공간
- **네트워크**: 다중 노드 테스트를 위한 네트워크 환경

### 2.2 필수 도구 설치
```bash
# CentOS/RHEL/Rocky Linux
sudo yum groupinstall "Development Tools"
sudo yum install cmake3 git python3-devel openssl-devel \
    readline-devel zlib-devel bzip2-devel

# Ubuntu/Debian
sudo apt-get update
sudo apt-get install build-essential cmake git python3-dev \
    libssl-dev libreadline-dev zlib1g-dev libbz2-dev
```

### 2.3 소스코드 다운로드 및 빌드
```bash
# 소스코드 클론
git clone https://github.com/apache/incubator-cloudberry.git
cd incubator-cloudberry

# 서브모듈 초기화
git submodule update --init --recursive

# 빌드 설정
./configure --with-python --enable-debug --enable-cassert

# 컴파일 (멀티코어 활용)
make -j$(nproc)

# 설치
make install
```

### 2.4 클러스터 초기화
```bash
# 환경 변수 설정
source /usr/local/cloudberry/cloudberry_path.sh

# 마스터 초기화
initdb -D /data/master

# 세그먼트 초기화 (다중 노드 환경)
gpseginstall -f hostfile_segments
gpinitsystem -c gpinitsystem_config
```

## 3. 커뮤니티 참여

### 3.1 공식 채널
- **GitHub**: https://github.com/apache/cloudberry
- **공식 웹사이트**: https://cloudberrydb.org/
- **Apache 메일링 리스트**: 
  - dev@cloudberry.apache.org (개발 논의)
  - commits@cloudberry.apache.org (커밋 알림)
  - issues@cloudberry.apache.org (이슈 트래킹)

### 3.2 커뮤니티 가입
```bash
# 메일링 리스트 구독
# dev 리스트 구독: dev-subscribe@cloudberry.apache.org로 빈 메일 발송
# 구독 해제: dev-unsubscribe@cloudberry.apache.org로 빈 메일 발송
```

### 3.3 Slack 및 Discord
- **CloudBerry Slack**: 실시간 커뮤니케이션
- **Apache Slack**: #cloudberry 채널
- **기술 토론**: 아키텍처, 성능, 버그 등 논의

### 3.4 정기 미팅
- **주간 개발자 미팅**: 매주 진행 상황 공유
- **월간 커뮤니티 미팅**: 로드맵 및 주요 결정사항 논의
- **분기별 기술 세미나**: 심화 기술 주제 발표

## 4. 기여 방법 및 영역

### 4.1 초보자를 위한 기여

#### 4.1.1 문서화 기여
- **README 개선**: 설치 및 사용법 문서화
- **API 문서**: 함수 및 클래스 문서 작성
- **튜토리얼**: 초보자를 위한 가이드 작성
- **번역**: 다국어 문서 지원

#### 4.1.2 테스트 및 버그 리포트
- **설치 테스트**: 다양한 환경에서의 설치 검증
- **기능 테스트**: 새로운 기능의 동작 검증
- **성능 테스트**: 벤치마크 및 성능 회귀 검사
- **버그 리포트**: 상세한 재현 단계와 함께 이슈 리포트

#### 4.1.3 코드 품질 개선
- **코딩 스타일**: 일관된 코드 스타일 적용
- **주석 개선**: 복잡한 로직에 대한 설명 추가
- **리팩토링**: 중복 코드 제거 및 구조 개선
- **경고 수정**: 컴파일러 경고 메시지 해결

### 4.2 중급 기여 영역

#### 4.2.1 SQL 기능 확장
- **새로운 SQL 함수**: 분석용 함수 추가
- **연산자 구현**: 특수 연산자 개발
- **데이터 타입**: 새로운 데이터 타입 지원
- **인덱스 최적화**: 인덱스 알고리즘 개선

#### 4.2.2 분산 처리 최적화
- **쿼리 최적화**: 분산 쿼리 플래너 개선
- **데이터 분산**: 파티셔닝 전략 개선
- **네트워크 최적화**: 노드 간 통신 효율화
- **로드 밸런싱**: 워크로드 분산 알고리즘

#### 4.2.3 저장소 엔진 개선
- **컬럼형 저장**: 압축 알고리즘 개선
- **캐시 시스템**: 메모리 캐시 최적화
- **I/O 최적화**: 디스크 접근 패턴 개선
- **백업/복구**: 분산 백업 시스템 개선

### 4.3 고급 기여 영역

#### 4.3.1 코어 아키텍처 개발
- **분산 트랜잭션**: ACID 속성 보장
- **동시성 제어**: 락 메커니즘 개선
- **장애 복구**: 고가용성 시스템 구축
- **메타데이터 관리**: 시스템 카탈로그 최적화

#### 4.3.2 성능 최적화
- **벡터화 실행**: SIMD 명령어 활용
- **병렬 처리**: 멀티스레딩 최적화
- **메모리 관리**: 메모리 풀 및 할당 최적화
- **컴파일러 최적화**: 코드 생성 및 최적화

#### 4.3.3 확장성 개발
- **플러그인 시스템**: 확장 모듈 아키텍처
- **외부 연동**: 다른 시스템과의 통합
- **클라우드 네이티브**: 컨테이너 및 오케스트레이션 지원
- **모니터링**: 시스템 모니터링 및 메트릭 수집

## 5. 개발 프로세스

### 5.1 이슈 선택 및 계획

#### 5.1.1 이슈 타입별 접근
- **good first issue**: 초보자 친화적 작업
- **bug**: 버그 수정 작업
- **enhancement**: 기능 개선
- **feature**: 새로운 기능 개발
- **documentation**: 문서 작업

#### 5.1.2 이슈 분석
```bash
# 이슈 라벨 확인
# - priority: P0(긴급), P1(높음), P2(보통), P3(낮음)
# - component: coordinator, segment, planner, executor
# - difficulty: easy, medium, hard
```

### 5.2 개발 워크플로우

#### 5.2.1 브랜치 전략
```bash
# 개발 브랜치 생성
git checkout main
git pull upstream main
git checkout -b feature/my-feature-name

# 개발 진행
# ... 코드 수정 ...

# 커밋
git add .
git commit -m "feat: add new distributed join algorithm"
```

#### 5.2.2 커밋 메시지 규칙
```
<type>(<scope>): <subject>

<body>

<footer>
```

**커밋 타입**:
- `feat`: 새로운 기능
- `fix`: 버그 수정
- `docs`: 문서 변경
- `style`: 코드 스타일 변경
- `refactor`: 리팩토링
- `test`: 테스트 추가/수정
- `chore`: 빌드/도구 변경

### 5.3 테스트 및 검증

#### 5.3.1 단위 테스트
```bash
# 단위 테스트 실행
make installcheck-world

# 특정 테스트 실행
pg_regress --inputdir=sql --outputdir=expected \
    --schedule=parallel_schedule test_name
```

#### 5.3.2 통합 테스트
```bash
# 분산 환경 테스트
gptest --setup-cluster
gptest --run-tests
gptest --cleanup
```

#### 5.3.3 성능 테스트
```bash
# TPC-H 벤치마크
pgbench -i -s 100 cloudberry_db
pgbench -c 10 -j 2 -T 300 cloudberry_db
```

### 5.4 Pull Request 제출

#### 5.4.1 PR 준비
```bash
# 코드 품질 검사
./tools/check_style.sh
make check

# 브랜치 정리
git rebase -i main
git push origin feature/my-feature-name
```

#### 5.4.2 PR 템플릿
```markdown
## 변경 사항 요약
- 주요 변경사항 설명

## 동기 및 배경
- 왜 이 변경이 필요한가?

## 구현 세부사항
- 어떻게 구현했는가?

## 테스트
- [ ] 단위 테스트 통과
- [ ] 통합 테스트 통과
- [ ] 성능 테스트 완료

## 체크리스트
- [ ] 코드 스타일 준수
- [ ] 문서 업데이트
- [ ] 커밋 메시지 규칙 준수
```

## 6. 리뷰 및 머지 프로세스

### 6.1 코드 리뷰
- **자동 검증**: CI/CD 파이프라인 통과
- **피어 리뷰**: 최소 2명의 커미터 승인
- **아키텍처 리뷰**: 주요 변경사항에 대한 설계 검토
- **성능 검토**: 성능 영향도 평가

### 6.2 피드백 대응
- **신속한 응답**: 24-48시간 내 응답
- **건설적 토론**: 기술적 근거를 바탕으로 한 토론
- **수정 반영**: 요청된 변경사항 적극 반영
- **학습 자세**: 리뷰어로부터 배우는 자세

### 6.3 머지 후 작업
- **배포 모니터링**: 머지 후 시스템 안정성 확인
- **문서 업데이트**: 필요시 추가 문서 작성
- **이슈 클로즈**: 관련 이슈 상태 업데이트
- **후속 작업**: 추가 개선사항 계획

## 7. 고급 기여자로 성장하기

### 7.1 코드 리뷰어 되기
- **기술적 전문성**: 특정 분야의 깊은 이해
- **코드 품질 감각**: 좋은 코드와 나쁜 코드 구분
- **커뮤니케이션**: 건설적 피드백 제공 능력
- **멘토링**: 신규 기여자 도움

### 7.2 커미터 권한 획득
- **지속적인 기여**: 6개월 이상 활발한 참여
- **코드 품질**: 높은 품질의 코드 기여
- **커뮤니티 참여**: 적극적인 커뮤니티 활동
- **Apache Way**: Apache 철학과 문화 이해

### 7.3 PMC(Project Management Committee) 멤버
- **프로젝트 거버넌스**: 프로젝트 방향성 결정 참여
- **릴리스 관리**: 버전 릴리스 책임
- **신규 커미터 승인**: 새로운 커미터 선정 참여
- **커뮤니티 리더십**: 커뮤니티 성장 주도

## 8. 유용한 리소스

### 8.1 기술 문서
- **Architecture Guide**: 시스템 아키텍처 이해
- **Developer Guide**: 개발자를 위한 상세 가이드
- **Performance Guide**: 성능 최적화 가이드
- **Troubleshooting Guide**: 문제 해결 가이드

### 8.2 개발 도구
- **IDE 설정**: VS Code, IntelliJ 설정 가이드
- **디버깅 도구**: GDB, Valgrind 사용법
- **프로파일링**: perf, gperftools 활용
- **정적 분석**: clang-static-analyzer, cppcheck

### 8.3 학습 자료
- **논문**: 분산 데이터베이스 관련 논문
- **컨퍼런스**: CloudBerry Summit, Apache conferences
- **블로그**: 기술 블로그 및 케이스 스터디
- **비디오**: 기술 세미나 및 튜토리얼

## 9. 주의사항 및 베스트 프랙티스

### 9.1 Apache Software Foundation 규칙
- **Apache License 2.0**: 모든 기여는 Apache 라이선스 하에
- **Developer Certificate of Origin**: 기여자 인증
- **No Proprietary Code**: 독점 코드 포함 금지
- **Export Control**: 수출 규제 준수

### 9.2 코드 품질 기준
- **테스트 커버리지**: 새 코드는 반드시 테스트 포함
- **문서화**: 공개 API는 반드시 문서화
- **성능**: 성능 저하를 일으키지 않도록 주의
- **보안**: 보안 취약점 방지

### 9.3 커뮤니티 에티켓
- **Respectful Communication**: 상호 존중하는 소통
- **Inclusive Behavior**: 포용적 행동
- **Constructive Feedback**: 건설적 피드백
- **Collaborative Spirit**: 협업 정신

---

CloudBerry Database는 빠르게 성장하는 오픈소스 프로젝트입니다. 여러분의 기여를 통해 함께 더 나은 분산 데이터베이스를 만들어 나갑시다!
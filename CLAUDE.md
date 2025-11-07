# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 저장소 개요

이 저장소는 **Obsidian 볼트**(지식 관리 시스템)이자 **GitHub Pages 사이트**로, PostgreSQL 개발 연구 및 기술 문서를 관리합니다.

### GitHub Pages 서비스
- **저장소**: https://github.com/assam258-5892/assam258-5892.github.io.git
- **공개 URL**: https://assam258-5892.github.io
- **목적**: PostgreSQL 개발 연구 및 기술 문서를 웹에서 공개

### 주요 콘텐츠
- PostgreSQL 오픈소스 기여 연구
- PostgreSQL 개발에서의 AI 활용 전략
- PostgreSQL 기반 엔터프라이즈 솔루션 분석
- 데이터베이스 기술 연구 및 실무 경험 공유

## 필수 관리 파일

다음 파일들은 GitHub Pages 서비스를 위해 **반드시 유지 관리**되어야 합니다:

### README.md
- 저장소 및 사이트 소개
- 방문자를 위한 첫 페이지
- 콘텐츠 개요 및 네비게이션 정보 포함
- 새로운 주요 문서 추가 시 목차에 링크 추가
- **.gitignore에 포함된 파일은 목차에 추가하지 않음**

### robots.txt
- 검색 엔진 크롤러 제어
- SEO 최적화를 위한 크롤링 규칙 정의
- 크롤링 제어가 필요한 페이지 추가 시 업데이트

### sitemap.xml
- 사이트 구조 및 페이지 목록
- 검색 엔진 인덱싱 최적화
- 새 문서 추가/수정 시 반드시 업데이트
- URL, 수정일(lastmod), 우선순위(priority) 정보 포함
- **.gitignore에 포함된 파일은 sitemap에 추가하지 않음**

## 문서 작성 규칙

### 언어 사용
- **기본 언어: 한국어**
- 모든 문서는 한국어로 작성
- Git 커밋 메시지는 한국어 또는 영어 가능
- 필요시 영문 요약 또는 번역 제공

### 파일 명명 규칙
- 설명적인 한국어 이름 사용
- 공백 포함 가능 (예: `PostgreSQL 개발 보고서.md`)
- Markdown 형식 (.md) 사용
- URL 친화적 영문 파일명도 고려 가능

### 문서 구조
- UTF-8 인코딩 필수
- 표준 Markdown 문법 준수
- 한국어 기술 문서 스타일 유지
- 제목, 부제목, 코드 블록, 링크 등 적절히 활용

### 메타데이터
- 문서 작성일, 수정일 기록 권장
- 카테고리, 태그 등 분류 정보 포함 고려
- SEO를 위한 설명문(description) 추가 권장

## 콘텐츠 주제

### PostgreSQL 개발 관련
- PostgreSQL 코어 기여 후보 항목
- 단독 개발자를 위한 AI 도구 활용 전략
- PostgreSQL 기술 이슈 및 번역 개선 사항
- 성능 최적화 및 튜닝 경험

### 경쟁 제품 분석
- PostgreSQL 기반 엔터프라이즈 솔루션 분석
- 제품 기능, 시장 포지셔닝, 기술 스펙 비교
- Oracle 대체 전략 및 마이그레이션 사례

### 기술 연구
- 데이터베이스 아키텍처 연구
- 고가용성 및 백업/복구 전략
- 클라우드 환경 PostgreSQL 운영
- AI 기반 데이터베이스 개발 워크플로우

## Obsidian 관련 규칙

### 설정 파일 관리
- `.obsidian/` 디렉토리는 Obsidian 앱의 설정 정보 저장
- 워크스페이스, 플러그인, 테마 설정 포함
- 필요한 경우가 아니면 수동 편집 지양
- GitHub에 커밋하지 않음 (.gitignore에 포함 권장)

### 동기화
- iCloud Drive를 통한 로컬 동기화
- Git을 통한 버전 관리 및 GitHub 동기화
- 충돌 발생 시 Git 버전을 우선으로 처리

### Obsidian 전용 기능
- 내부 링크 `[[문서명]]` 사용 가능
- GitHub Pages에서는 표준 Markdown 링크로 변환 필요
- 플러그인 의존성 최소화 권장

## Git 및 GitHub Pages 관리

### 기본 워크플로우
```bash
# 변경 사항 확인
git status

# 차이점 확인
git diff

# 스테이징
git add .

# 커밋
git commit -m "설명적인 커밋 메시지"

# GitHub에 푸시
git push origin main
```

### 커밋 메시지
- 한국어 또는 영어 사용 가능
- 변경 내용을 명확히 설명
- 예시:
  - "PostgreSQL 기여 후보 목록 추가"
  - "Add competitive analysis for eXperDB"
  - "Update sitemap.xml with new articles"

### GitHub Pages 배포
- `main` 브랜치의 변경사항이 자동으로 배포됨
- 배포 완료까지 수 분 소요 가능
- https://assam258-5892.github.io 에서 변경사항 확인
- 배포 상태는 GitHub 저장소의 Actions 탭에서 확인

## AI 도구 활용 컨텍스트

이 저장소는 **대규모 PostgreSQL 프로젝트를 수행하는 단독 개발자**의 연구 자료이자 기술 블로그입니다.

### 핵심 배경
- AI 도구 없이는 단독으로 PostgreSQL 개발이 불가능
- Claude Max, Copilot Pro 등 프리미엄 AI 구독 필요성 강조
- PostgreSQL 코어 기여 기회 식별
- 실무 경험과 연구를 커뮤니티와 공유

### 문서의 목적
- PostgreSQL 코어 기여 계획 수립
- 엔터프라이즈 PostgreSQL 기술 동향 추적
- AI 도구 활용 사례 공유
- 한국어 PostgreSQL 기술 콘텐츠 생산

## 새 문서 작성 시 유의사항

### 콘텐츠 작성
- 한국어로 작성하여 일관성 유지
- 기술적 정확성과 심층 분석 중시
- 실무 적용 가능한 구체적 정보 포함
- 필요시 영문 용어는 한글 설명과 병기

### 웹 공개 고려사항
- 민감한 정보(회사명, 프로젝트명 등) 노출 주의
- 저작권 및 라이선스 명시
- 외부 링크 및 이미지는 유효성 확인
- 코드 예제는 실행 가능하고 검증된 내용만 포함

### SEO 최적화
- 명확하고 설명적인 제목 사용
- 적절한 제목 계층 구조(H1, H2, H3) 유지
- 핵심 키워드를 자연스럽게 포함
- 내부 링크를 통한 문서 간 연결
- 메타 설명 추가 권장

### 문서 추가 후 작업
- README.md에 새 문서 링크 추가
- sitemap.xml 업데이트
- 필요시 robots.txt 수정
- **.gitignore에 포함된 파일은 README.md와 sitemap.xml에 추가하지 않음**

### .gitignore 필터 규칙
- .gitignore에 명시된 파일/디렉토리는 비공개 문서로 간주
- 현재 필터 대상: `.obsidian/`, `CLAUDE.md`
- 비공개 문서는 웹에 노출되지 않아야 함
- README.md 목차와 sitemap.xml에서 제외

## 문제 해결

### 파일 동기화 충돌
- Git과 iCloud 간 충돌 발생 가능
- Git pull 우선 수행 후 작업
- 충돌 발생 시 Git 버전을 기준으로 해결

### GitHub Pages 빌드 실패
- GitHub Actions 로그 확인
- Markdown 문법 오류 검토
- 파일 인코딩(UTF-8) 확인

### Obsidian 링크 호환성
- `[[문서명]]` → `[문서명](문서명.md)` 형식으로 변환
- 상대 경로 링크 사용
- GitHub Pages에서 링크가 작동하는지 확인

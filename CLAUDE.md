# CLAUDE.md

Claude Code (claude.ai/code) 작업 가이드.

## 저장소 개요

Obsidian 볼트의 `/출판` 폴더 = GitHub Pages 사이트.
PostgreSQL 개발 연구 및 기술 문서 공개용.

- **저장소**: https://github.com/assam258-5892/assam258-5892.github.io.git
- **공개 URL**: https://assam258-5892.github.io

## 동기화 구조

```
Obsidian 볼트 (iCloud 동기화, 무료)
│
├── /출판          ← Git + GitHub Pages (공개)
│   ├── CLAUDE.md
│   ├── README.md
│   ├── robots.txt
│   ├── sitemap.xml
│   └── /PostgreSQL
│
├── /개발          ← iCloud만 (비공개)
├── /규칙          ← iCloud만 (비공개)
├── /기여          ← iCloud만 (비공개)
├── /제품          ← iCloud만 (비공개)
├── /업무          ← iCloud만 (비공개)
└── /연구          ← iCloud만 (비공개)
```

**워크플로우:**
```
Obsidian 작성 → iCloud 동기화 → Git 푸시 → GitHub Pages 자동 배포
```

**비용:** $0 (iCloud가 Obsidian Sync $4/월 대체, GitHub Pages가 Publish $8/월 대체)

## .gitignore (블랙리스트 방식)

```
.obsidian/
.DS_Store
*.tmp
*~
```

`/출판` 폴더 내 모든 파일이 GitHub Pages에 배포됨.

## 문서 작성 규칙

- **언어**: 한국어 (커밋 메시지는 한국어/영어 가능)
- **파일명**: 한국어 가능, 공백 가능 (예: `PostgreSQL 개발 보고서.md`)
- **인코딩**: UTF-8
- **링크**: Obsidian `[[문서명]]` → GitHub용 `[문서명](문서명.md)` 변환 필요

## 문서 추가 후 작업

1. `README.md` 목차에 링크 추가
2. `sitemap.xml` 업데이트 (URL, lastmod, priority)

## 웹 공개 주의사항

- 민감 정보 (회사명, 프로젝트명) 노출 주의
- `/출판`에 넣는 순간 공개 각오

## 문제 해결

| 문제 | 해결 |
|------|------|
| Git-iCloud 충돌 | Git pull 먼저, Git 버전 기준 해결 |
| GitHub Pages 빌드 실패 | Actions 로그 확인, Markdown 문법/UTF-8 검토 |
| Obsidian 링크 깨짐 | `[[문서명]]` → `[문서명](문서명.md)` 변환 |

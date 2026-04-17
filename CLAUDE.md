# CLAUDE.md — JD 분석기

Claude가 이 프로젝트를 작업할 때 참고하는 컨텍스트 파일입니다.

---

## 프로젝트 개요

이력서를 등록하고 채용 공고(JD)를 입력하면 매칭 분석·어필 포인트·서류 통과 가이드를 자동 생성하는 웹 도구.  
단일 HTML 파일로 동작하며 빌드 과정 없이 브라우저에서 바로 실행됩니다.

**라이브 URL:** https://jaycy-js.github.io/JD-analyzer  
**레포:** https://github.com/jaycy-JS/JD-analyzer

---

## 파일 구조

```
JD-analyzer/
├── index.html          # 앱 전체 (HTML + CSS + JS 단일 파일)
├── CLAUDE.md           # 이 파일
├── README.md           # 사용자용 문서
├── docs/
│   └── adr.md          # Architecture Decision Records
└── .gitignore
```

---

## 기술 스택

| 역할 | 선택 | 이유 |
|------|------|------|
| 프레임워크 | Vanilla HTML/CSS/JS | 빌드 불필요, 단일 파일 유지 |
| AI 분석 | Anthropic Claude API (`claude-sonnet-4-20250514`) | 고품질 한국어 분석 |
| PDF 파싱 | pdfjs-dist 3.11.174 (jsdelivr) | 브라우저 네이티브 PDF 처리 |
| DOCX 파싱 | mammoth 1.8.0 (jsdelivr) | 경량 DOCX 텍스트 추출 |
| URL 크롤링 | corsproxy.io | CORS 우회 (불안정, ADR-005 참고) |
| 저장소 | sessionStorage | 개인정보 보호 (탭 닫으면 소멸) |
| 호스팅 | GitHub Pages | 무료, 레포 연동 |

---

## 주요 제약사항

- **단일 파일 원칙**: CSS·JS를 별도 파일로 분리하지 않음. index.html 하나로 유지.
- **빌드 없음**: npm/webpack 사용 안 함. CDN 라이브러리만 허용.
- **DB 없음**: 서버·DB 연동 없음. 모든 데이터는 sessionStorage.
- **max_tokens: 1000**: Claude API 호출 시 고정값 사용.
- **한국어 우선**: UI·분석 결과 모두 한국어.

---

## 4단계 뷰 구조

```
v1 (프로필 등록) → v2 (JD 입력 + 히스토리) → v3 (분석 결과) → v4 (전체 비교)
```

- `goV(n)` 함수로 뷰 전환, stepbar 상태 동시 업데이트
- 분석 결과는 `analyses[]` 배열에 누적, sessionStorage에 직렬화 저장 (최대 10개)

---

## 분석 JSON 스펙

Claude API가 반환하는 JSON 필드명:

```json
{
  "score": 0-100,
  "title": "한줄 적합도",
  "desc": "종합 평가 2문장",
  "position": "회사명·포지션명",
  "positioning": "JD맞춤 포지셔닝 한 줄",
  "kw": [{"w": "키워드", "l": "high|mid|low"}],
  "matches": ["매칭 강점 4개"],
  "gaps": ["갭 3개"],
  "appeals": [{"t": "제목", "b": "내용", "p": "서류용 문장"}],
  "warnings": [{"t": "제목", "d": "설명"}],
  "guides": [{"t": "제목", "d": "팁"}],
  "qs": ["예상 면접 질문 4개"]
}
```

---

## 작업 시 주의사항

- CSS 변수(`--accent`, `--bg` 등)는 상단 `:root`에 정의됨. 하드코딩 금지.
- 클래스명은 짧은 약어 스타일 (`.bp`, `.bg`, `.ae`, `.ao` 등). 기존 패턴 유지.
- 라이브러리 로드 실패는 `window._pdfFailed`, `window._mammothFailed` 플래그로 감지.
- 새 기능 추가 시 ADR(`docs/adr.md`)에 결정 사항 기록.
- QA 결과는 `docs/qa-log.md`에 누적 기록.

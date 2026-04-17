# ADR — Architecture Decision Records

JD 분석기 프로젝트의 주요 의사결정 기록입니다.  
새로운 결정이 생길 때마다 이 파일을 업데이트합니다.

---

## ADR-001 · 단일 HTML 파일 구조 채택

**날짜:** 2026-04-17  
**상태:** ✅ Accepted

**컨텍스트**  
팀 내 비개발자도 파일을 공유·사용할 수 있어야 하고, GitHub Pages 배포를 전제로 함.

**결정**  
HTML·CSS·JS를 `index.html` 하나에 통합. 빌드 도구 없음.

**결과**  
- 장점: 배포 단순, 의존성 없음, 파일 하나로 공유 가능  
- 단점: 파일이 길어질수록 유지보수 난이도 상승. 500줄 초과 시 섹션 주석으로 구분 필수

---

## ADR-002 · 데이터 저장소로 sessionStorage 선택

**날짜:** 2026-04-17  
**상태:** ✅ Accepted

**컨텍스트**  
이력서·경력기술서는 민감한 개인정보. 서버 전송·영구 저장 시 보안 이슈.

**결정**  
`sessionStorage` 전용 사용. 탭 닫으면 자동 소멸. DB·서버 저장 없음.  
`localStorage`는 영구 저장이므로 사용 금지.

**결과**  
- 장점: 개인정보 보호, 서버 인프라 불필요  
- 단점: 새 탭 열거나 브라우저 재시작 시 프로필 재입력 필요

---

## ADR-003 · PDF·DOCX 파싱 라이브러리 선택

**날짜:** 2026-04-17  
**상태:** ✅ Accepted

**컨텍스트**  
브라우저 환경에서 파일을 서버 전송 없이 로컬 파싱해야 함.

**결정**  
- PDF: `pdfjs-dist@3.11.174` (jsdelivr) — 브라우저 네이티브, 워커 분리
- DOCX: `mammoth@1.8.0` (jsdelivr) — 경량, `extractRawText` API 단순

**기각된 대안**  
- cdnjs: mammoth 미등록, PDF.js URL 형식 불일치로 기각 (QA에서 발견)
- 서버사이드 파싱: 아키텍처 원칙(서버 없음) 위반

**결과**  
- 스캔된 이미지 PDF는 텍스트 추출 불가 → 사용자에게 명시적 안내
- 라이브러리 로드 실패 시 `window._pdfFailed` / `window._mammothFailed` 플래그로 감지 후 fallback 메시지 표시

---

## ADR-004 · Claude API 호출을 클라이언트에서 직접 수행

**날짜:** 2026-04-17  
**상태:** ✅ Accepted (임시)

**컨텍스트**  
서버 없는 구조에서 AI 분석을 어떻게 수행할지 결정 필요.

**결정**  
브라우저에서 `fetch`로 Anthropic API 직접 호출.  
API Key는 클라이언트에 노출되지 않음 (Claude.ai 환경의 내장 핸들링 활용).

**결과**  
- 장점: 서버 불필요, 구조 단순  
- ⚠️ 위험: 일반 배포 시 API Key 클라이언트 노출 문제 발생 가능  
- 향후 사용자 수 증가 시 백엔드 프록시 레이어 검토 필요 (ADR 업데이트 예정)

---

## ADR-005 · JD URL 크롤링에 corsproxy.io 사용

**날짜:** 2026-04-17  
**상태:** ⚠️ Accepted with Risk → 🔴 기능 축소 결정 (2026-04-17 QA 결과)

**컨텍스트**  
JD URL을 직접 fetch하면 CORS 정책으로 대부분 차단됨. 서버가 없으므로 프록시 필요.

**결정**  
`https://corsproxy.io/?{encodedURL}` 패턴으로 CORS 우회.  
실패 시 "직접 붙여넣기" fallback으로 자동 전환.

**QA 결과 (2026-04-17)**  
주요 채용 플랫폼 전수 테스트 결과, **모두 차단** 확인:

| 플랫폼 | 결과 | 원인 |
|--------|------|------|
| Wanted | ❌ | Cloudflare Bot 감지, JS SPA |
| JobKorea | ❌ | User-Agent + 세션 쿠키 |
| Saramin / Jumpit | ❌ | Anti-scraping 헤더 |
| LinkedIn | ❌ | 로그인 필수 |
| Rocketpunch | ⚠️ | 부분적으로 가능 |
| Google Docs / Notion | ❌ | 인증 필요 |

**결과로 인한 변경**  
- UI에서 URL 입력을 `<details>` 접힘 처리 → Secondary 기능으로 격하  
- "실험적 · 대부분 차단됨" 레이블 명시  
- 직접 붙여넣기를 Primary UI로 격상  
- 향후 서버사이드 크롤러 또는 브라우저 익스텐션 방식 검토 예정

---

## ADR-006 · GitHub Pages로 호스팅

**날짜:** 2026-04-17  
**상태:** ✅ Accepted

**컨텍스트**  
팀원들이 URL 하나로 접근할 수 있어야 함. 인프라 관리 비용 최소화.

**결정**  
`jaycy-JS/JD-analyzer` 레포의 GitHub Pages 활성화.  
`main` 브랜치 push → 자동 배포.

**결과**  
- 배포 URL: https://jaycy-js.github.io/JD-analyzer  
- 무료, CI/CD 불필요, push만으로 반영  
- 단점: 커스텀 도메인 없음 (필요 시 별도 ADR 작성)

---

## ADR-007 · QA 서브 에이전트 분리 전략

**날짜:** 2026-04-17  
**상태:** 🔄 Proposed

**컨텍스트**  
기능 추가가 누적될수록 수동 QA는 비효율. 별도 QA 에이전트가 독립적으로 검증해야 함.

**결정 (안)**  
- `docs/qa-plan.md`: 테스트 케이스 정의 (사람이 작성)
- Claude 서브 에이전트: qa-plan을 읽고 각 케이스 실행 후 `docs/qa-log.md`에 결과 기록
- 메인 에이전트와 QA 에이전트 역할 분리 — 메인은 기능 구현, QA는 검증만 담당

**미결 사항**  
- QA 에이전트 트리거 시점 (PR 시 자동 / 수동 호출)
- 브라우저 자동화 도구 연동 여부 (Playwright 등)

---

*새 결정이 생길 때마다 이 파일에 ADR-00N 형식으로 추가합니다.*

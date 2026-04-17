# QA Log

QA 결과를 누적 기록합니다. 날짜 내림차순.

---

## QA-002 · URL 크롤링 플랫폼별 가능 여부

**날짜:** 2026-04-17  
**담당:** Claude (메인 에이전트)  
**대상:** corsproxy.io 기반 JD URL / 이력서 URL 크롤링

### 테스트 결과

| 플랫폼 | 결과 | 차단 원인 |
|--------|------|-----------|
| Wanted | ❌ 차단 | Cloudflare Bot 감지, JS 렌더링(React SPA) |
| JobKorea | ❌ 차단 | User-Agent 검사 + 세션 쿠키 필요 |
| Saramin | ❌ 차단 | Anti-scraping 헤더 검사 |
| Jumpit | ❌ 차단 | Saramin 계열, 동일 정책 |
| LinkedIn | ❌ 차단 | 로그인 필요, 강력한 Bot 방어 |
| Rocketpunch | ⚠️ 부분 | Bot 방어 약함, 단순 HTML 구조 |
| Google Docs (이력서) | ❌ 차단 | 인증 필요, 공개 링크도 차단 |
| Notion (이력서) | ❌ 차단 | CORS 완전 차단 |

### 근본 원인

1. **JS 렌더링(SPA)**: 원티드 등 대부분 React 기반. 프록시가 HTML을 가져와도 JD 본문은 JS 실행 후 렌더링되므로 텍스트 없음.
2. **Bot 방어**: Cloudflare / 자체 방어로 프록시 IP 대역 차단.
3. **인증 필요**: LinkedIn, Google Docs, Notion은 로그인 없이 접근 불가.

### 결정

- **URL 입력 기능**: "실험적 기능" 레이블 추가, 낮은 기대치 설정
- **안내 문구 개선**: 특정 도메인 감지 시 맞춤 경고 메시지 표시
- **직접 붙여넣기를 Primary UI로 격상**: 섹션 순서 변경
- ADR-005 업데이트 필요

---

## QA-001 · 파일 읽기 라이브러리 로드 검증

**날짜:** 2026-04-17  
**담당:** Claude (메인 에이전트)  
**대상:** PDF.js, mammoth.js CDN 로드

### 발견된 버그

| # | 증상 | 원인 | 심각도 |
|---|------|------|--------|
| B-001 | PDF 업로드 시 `pdfjsLib is not defined` | cdnjs URL 오류 (패키지명 불일치) | 🔴 Critical |
| B-002 | DOCX 업로드 시 `mammoth is not defined` | mammoth이 cdnjs에 미등록 (404) | 🔴 Critical |
| B-003 | 스캔 PDF 업로드 시 빈 텍스트로 통과 | 텍스트 추출 결과 검증 없음 | 🟡 Medium |
| B-004 | .jpg 등 비지원 파일 업로드 시 오류 메시지 불명확 | 파일 확장자 분기 없음 | 🟡 Medium |
| B-005 | 10MB 이상 파일 업로드 시 브라우저 멈춤 | 파일 사이즈 제한 없음 | 🟡 Medium |

### 수정 내용

- CDN을 cdnjs → **jsdelivr** 로 전환 (pdfjs-dist@3.11.174, mammoth@1.8.0)
- `onerror` 핸들러 추가 → `window._pdfFailed`, `window._mammothFailed` 플래그
- `readPDF()`, `readDOCX()` 에 라이브러리 가용성 체크 추가
- 스캔 PDF 감지 (추출 텍스트 30자 미만 시 안내)
- 비지원 확장자 즉시 차단 + 명확한 에러 메시지
- 10MB 파일 사이즈 제한 추가

### 상태: ✅ 모두 수정됨 (17/17 체크 통과)

---

## QA 서브 에이전트 설계 (예정)

> 현재는 메인 에이전트가 QA를 겸하고 있음. 기능이 확대되면 분리 필요.

**역할 분리 계획**
- 메인 에이전트: 기능 구현
- QA 에이전트: `docs/qa-plan.md` 읽기 → 케이스 실행 → 이 파일에 결과 기록

**QA 에이전트 트리거 시점 (미결)**
- PR 생성 시 자동 실행
- 수동 호출 (`@QA 검증해줘`)
- 주요 기능 변경 시 자동

→ ADR-007 참고

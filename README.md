# JD 분석기 (JD Analyzer)

이력서를 등록하고 채용 공고(JD)를 입력하면 **매칭 점수 · 어필 포인트 · 서류 통과 가이드**를 자동으로 분석해주는 웹 도구입니다.

**라이브 데모 →** `https://jaycy-JS.github.io/JD-analyzer`

---

## 주요 기능

| 기능 | 설명 |
|------|------|
| 프로필 등록 | PDF / DOCX / TXT 파일 업로드 또는 URL · 직접 입력 |
| JD 분석 | URL 자동 크롤링 또는 텍스트 붙여넣기 |
| 분석 결과 | 매칭 점수, JD맞춤 포지셔닝, 어필 포인트, 주의사항, 서류 가이드, 예상 면접 질문 |
| 멀티 분석 | 여러 JD를 연속 분석 후 점수 순 비교 |
| 개인정보 보호 | 세션 전용 저장 (탭 닫으면 자동 삭제, 서버 저장 없음) |

---

## 로컬 실행

별도 설치 불필요. 파일을 브라우저로 열면 바로 실행됩니다.

```bash
git clone https://github.com/jaycy-JS/JD-analyzer.git
cd jd-analyzer
open index.html   # macOS
# 또는 index.html을 브라우저로 드래그
```

---

## GitHub Pages 배포

레포 → Settings → Pages → Branch: `main` / `/ (root)` → Save

저장 후 약 1분 뒤 `https://jaycy-JS.github.io/JD-analyzer` 에서 접근 가능합니다.

---

## 기술 스택

- Vanilla HTML / CSS / JS (빌드 불필요)
- Anthropic Claude API (`claude-sonnet-4-20250514`)
- PDF.js — PDF 텍스트 추출
- Mammoth.js — DOCX 텍스트 추출
- corsproxy.io — URL 크롤링 CORS 우회

---

## 개발 로드맵

- [ ] 자기소개서 초안 자동 생성 (어필 포인트 기반)
- [ ] 분석 결과 PDF 다운로드
- [ ] 온보딩 모달 (첫 실행 안내)
- [ ] 다국어 지원 (영문 JD 분석)
- [ ] 모바일 UX 개선

---

## 라이선스

MIT

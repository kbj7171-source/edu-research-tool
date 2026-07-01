# ITCEN VisionCenter 교육기획 AI Agent — 인수인계 문서

> 다른 사람이 이 프로젝트를 이어서 작업할 수 있도록 정리한 문서입니다.
> 비개발자도 따라올 수 있게 "무엇을 / 왜 / 어디를 고치면 되는지" 중심으로 적었습니다.
> 최종 갱신: 2026-06-17

---

## 1. 한 줄 요약

**교육 주제 키워드를 입력하면, Gemini가 웹 검색으로 트렌드·타사 사례·커리큘럼 등을 조사해
보고서로 정리하고, 그 결과로 품의서·계획서·설문·발표자료까지 만들어 주는 웹 도구**입니다.
파일 하나(`index.html`)로 동작합니다.

- 목적: 교육 기획 담당자가 "근거 있게" 기획하고 결재까지 빠르게 (배경은 `PRD_교육기획근거도우미.md`)
- 형태: HTML 파일 1개 (서버·빌드·설치 없음)
- AI: Google Gemini API (무료 등급, 웹 검색 그라운딩 사용)
- 배포: GitHub Pages (정적 호스팅)
- 공개 URL: <https://kbj7171-source.github.io/edu-research-tool/>
- 저장소: <https://github.com/kbj7171-source/edu-research-tool>

---

## 2. 지금 상태

- ✅ GitHub Pages 배포 중 (위 공개 URL)
- ✅ 누구나 링크 접속 후 **본인 Gemini 무료 키**를 넣고 사용(키·결과는 그 사람 브라우저에만 저장)
- ⚠️ **실제 키로 한 브라우저 end-to-end 테스트는 아직 미완** — API 호출 기능(조사/키워드/문서생성)은
  코드 검증(문법·요소 존재)만 마친 상태. 실제 키로 한 번 돌려보는 것을 권장.

---

## 3. 주요 기능 (현재)

**조사**
- 교육 주제 + 옵션(조사 항목 체크 13종, 출력타입/범위/분량 + 기간/규모/직급/규모 드롭다운, 자유 입력)
- **실시간 스트리밍 출력**(글이 흐르듯), 혼잡 시 자동 재시도 + 보조 모델 전환
- **연관 키워드 추천**(주제 입력 후 버튼 → 칩 클릭으로 주제 교체)

**결과 활용**
- 보고서 카드 + 검색 출처(번호·사이트 표시) + 시간배분 막대그래프 + 커리큘럼 표(세부 학습내용/목표)
- **복사 / Markdown(.md) / Word(.doc) / 인쇄·PDF**
- **결과 공유 링크**(보고서를 URL 해시 `#r=`에 담아 백엔드 없이 공유)
- **이어서 질문하기**(멀티턴, 맥락 유지)

**AI 추가 문서 생성 (버튼 = 토글: 누르면 생성, 다시 누르면 닫힘/다시 열림)**
- 📊 발표 슬라이드 개요 / 📋 교육 품의서 / 🗂 교육 운영 계획서 / 📝 설문(사전·사후)
- 🎤 임원 예상질문 & 방어 답변 / 🥊 레드팀 허점 검토
- 닫은 문서는 인쇄·다운로드·공유에서 제외됨

**부가**
- 💰 예산 계산기(직접 입력 + "AI로 단가 추정해 채우기", 숫자 바꾸면 합계 자동 재계산)
- 📈 ROI·기대효과 산출(예산=투자 대비 기대 성과, "AI로 향상률 추정" + 직접 수정, ROI·순효과·회수기간 자동 계산, 예산 계산기 값 연동)
- 🕘 이전 조사 기록(자동 저장 최근 20건, JSON 내보내기/가져오기)
- 조건 프리셋 저장/불러오기
- 📄 보고서 표지 설정(회사·부서명/작성자/로고 → 인쇄·PDF·Word 표지에 표시)
- 🌙 다크모드(토스 스타일 테마: 라이트=흰 배경+블루, 다크=블랙+블루)
- ❓ 첫 사용 가이드(첫 방문 자동 표시 + 버튼)

---

## 4. 기술 구성 (의도적으로 단순하게)

| 항목 | 내용 | 왜 이렇게 했나 |
|------|------|----------------|
| 파일 | `index.html` 하나 (+ 로고 png) | 비개발자가 환경설정 없이 바로 쓰고 배포 |
| 의존성 | 없음 (외부 라이브러리 X) | 깨질 부분 최소화 |
| AI 호출 | 브라우저에서 Gemini API 직접 fetch (스트리밍) | 서버 없이 동작 |
| 웹 검색 | Gemini `google_search` 그라운딩 | 최신 정보 + 출처 확보(환각 줄이기) |
| API 키 | 코드에 안 박음 → 사용자가 입력 → `localStorage` | 키 유출 방지. 공개 배포해도 안전 |
| 저장 | 전부 `localStorage`(브라우저 안) | 서버·DB 없음. PC 옮길 땐 기록 JSON 내보내기/가져오기 |

> **보안 원칙**: API 키 등 비밀값은 절대 코드/저장소에 넣지 않습니다. 키는 사용자 브라우저
> `localStorage`(`gemini_api_key`)에만 저장되고 외부로 전송되지 않습니다.

### 사용하는 localStorage 키
`gemini_api_key`, `edu_theme`, `edu_company`, `edu_author`, `edu_logo`,
`edu_history_v1`(기록), `edu_presets_v1`(프리셋), `edu_seen_guide`(가이드 본 적 있음)

---

## 5. 파일 목록 / 저장소 구성

**저장소(GitHub)에 올라가는 = 웹사이트 구성 파일** (`.gitignore` 화이트리스트로 이것만 추적)
- **`index.html`** — 실제 도구 (핵심)
- `01_ITCEN_Logo_Basic.png` — 상단 로고
- `HANDOFF.md` — (이 문서)
- `.gitignore` — 추적 대상 화이트리스트

**로컬에만 있고 저장소엔 안 올라감(내부 자료)**
- `PRD_교육기획근거도우미.md`, `00_용어설명.md`~`09_E_CI.md`, `hanbit-prd-interview.skill` 등

> `프로젝트폴더`가 곧 이 GitHub 저장소의 로컬 작업본입니다. 강의자료·PRD는 `.gitignore`로
> 가려져 GitHub에 공개되지 않습니다.

---

## 6. 코드 구조 (`index.html` 내부)

`<style>`(디자인·테마) + `<body>`(화면) + `<script>`(동작) 세 부분.

### 6-1. 디자인/테마
- 색은 전부 **CSS 변수**(`:root` 라이트, `:root[data-theme="dark"]` 다크). 색을 바꾸려면 이 두 블록만 수정.
- 인쇄(`@media print`)는 다크여도 항상 라이트로 강제. 모바일(`@media max-width:640px`) 대응.

### 6-2. 자바스크립트 핵심 (여기를 고치면 동작이 바뀜)

| 위치 | 역할 |
|------|------|
| `const MODELS = [...]` | 쓸 Gemini 모델 목록(혼잡 시 순서대로 전환). **모델명 바뀌면 여기만 수정** |
| `callGeminiStream(...)` | 실제 스트리밍 API 호출(SSE 파싱). `useSearch=false`면 검색 끔(키워드 추천용) |
| `runGemini(apiKey, contents, onText, useSearch)` | 재시도+보조모델 전환 래퍼. 모든 호출이 여기 통함 |
| `chunkText/chunkCites` | 스트리밍 조각에서 본문/출처 추출 |
| `const ITEMS = [...]` | **조사 항목 목록**(체크박스 자동 생성) |
| `buildPrompt(...)` | 선택값을 모아 조사 프롬프트 조립 (각 `~Rule`로 표/그래프/번호목록 지시) |
| `renderMarkdown(md)` | 마크다운 → 보고서 HTML(카드·표·불릿). 표 셀의 `<br>`만 줄바꿈 허용 |
| `renderChart/renderSources/buildCoverHtml` | 막대그래프 / 출처목록 / 인쇄 표지 |
| `finalizeReport(text,cites,topic,industry,save)` | 보고서 렌더+표지+출처+도구노출+기록저장 (조사/기록불러오기/공유 공통) |
| `research()` | 조사하기 클릭 전체 흐름(스트리밍 렌더 → finalize) |
| `askFollowup(q, label, opts)` | 멀티턴 질문/문서 생성. `opts.key` 있으면 토글 가능 문서 |
| `toggleDoc(key, btn, prompt, label)` | 추가 문서 버튼 열고/닫기. `docBlocks` 맵으로 추적 |
| `exportMarkdown/exportDoc/downloadFile` | .md / .doc 내보내기 (닫힌 문서 제외) |
| 기록: `loadHistory/saveHistoryRecord/renderHistory/loadHistoryItem` + export/import | localStorage 기록 |
| 프리셋: `snapshotOptions/applyOptions/renderPresetOptions` | 옵션 조합 저장 |
| 예산: `computeBudget/estimateBudget/budgetSummaryText` | 계산 + AI 단가 추정 |
| 공유: `b64encode/b64decode/loadSharedReport` + shareBtn | URL 해시 공유 |
| 표지: `buildCoverHtml` + 로고/회사/작성자 저장 | 인쇄 표지 |
| 가이드: `openGuide/closeGuide` | 온보딩 모달 |

### 6-3. 조사 흐름
```
[조사하기] → buildPrompt() → runGemini(스트리밍, onText로 실시간 렌더)
          → finalizeReport(): 표/그래프/출처/표지 정식 렌더 + 기록 저장
[추가 문서 버튼] → toggleDoc() → (없으면) askFollowup(): 보고서 맥락 + 지시 프롬프트로 생성
```

---

## 7. 자주 하는 수정 (레시피)

- **조사 항목 추가/삭제**: `ITEMS` 배열에 `{ id, label, text, def }` 한 줄. 특수 형식은 `buildPrompt`의 `~Rule` 참고.
- **Gemini 모델 변경**: `const MODELS = [...]` 모델명 교체.
- **프롬프트 말투/형식**: `buildPrompt`의 return 템플릿(`[출력 형식]`/`[정확성 규칙]`) 수정.
- **추가 문서 버튼 추가**: 버튼 HTML + 요소 ref + `genButtons()` 목록에 추가 + `toggleDoc("새키", 버튼, "프롬프트", "라벨")` 배선.
- **테마 색 변경**: `:root` / `:root[data-theme="dark"]` 변수값만 수정.
- **표 셀 줄바꿈**: 프롬프트에서 셀 안에 `<br>` 쓰게 지시 → `inline()`이 `<br>`만 살려줌(다른 태그는 escape).

---

## 8. 배포 / 업데이트 방법 (중요 — 어제와 달라짐)

**이제 `프로젝트폴더`가 GitHub 저장소에 직접 연결돼 있습니다.** 웹에서 업로드할 필요 없이
이 폴더에서 바로 푸시하면 됩니다.

1. `프로젝트폴더`의 `index.html`(또는 로고 등) 수정
2. 터미널(Git Bash/PowerShell)에서:
   ```bash
   git add index.html
   git commit -m "수정 내용 설명"
   git push
   ```
   (Claude Code에게 "수정하고 배포해줘"라고 해도 됨)
3. 1~2분 뒤 공개 URL을 **Ctrl+F5**(강력 새로고침)로 확인

- 새 파일을 웹사이트에 포함시키려면 `.gitignore`의 화이트리스트(`!/파일명`)에 추가해야 함.
- 배포 후 반영 확인: 공개 URL을 새로고침했을 때 변경이 보이면 완료.

---

## 9. 알려진 제약 / 주의사항

| 항목 | 내용 |
|------|------|
| **API 키 각자 발급** | Gemini 무료. 키는 `AIza`로 시작(OpenAI `sk-...`와 혼동 주의). 발급: <https://aistudio.google.com/app/apikey> |
| **무료 한도/혼잡** | 분당 요청수 제한/일시 혼잡 있음 → 자동 재시도·보조모델 전환으로 완화. 계속 막히면 잠시 후 재시도 |
| **환각** | AI가 사실을 지어낼 수 있음. 본문 모델 링크는 클릭 비활성화, **실제 링크는 "검색 출처"에서만**. 회사 사례·수치는 재확인 권장 |
| **공유 링크 길이** | 아주 긴 보고서는 링크화 불가(안내 뜸) → Word/PDF/.md로 공유 |
| **기록은 브라우저별** | 기기·브라우저마다 따로. 옮길 땐 기록 **JSON 내보내기 → 가져오기** |
| **로고 다크모드** | 로고 글자가 검정이라 다크모드에선 흰 박스를 깔아 보이게 처리함 |
| **모델명 변동** | 바뀌면 `MODELS` 수정. 오류 메시지는 화면에 그대로 노출됨 |

---

## 10. 작업 이력 (요약)

**어제까지(v1)**: PRD → MVP → "키워드로 조사" 방향 전환 → Gemini 직접 연결(무료+웹검색) →
조사 항목/필터/드롭다운 → 보고서 카드 레이아웃·인쇄 → 커리큘럼표/시간그래프/뉴스링크 →
혼잡 재시도·보조모델 → 링크 환각 차단 → **GitHub Pages 배포**.
(중간에 OpenAI로 잠깐 바꿨다가 무료 문제로 Gemini 복귀 — 현재 코드에 OpenAI 흔적 없음)

**오늘(v2) 추가**:
1. 자유 입력(추가 요청사항) 칸
2. `프로젝트폴더`를 저장소에 직접 연결(배포 방식이 web 업로드 → git push 로 변경)
3. 실시간 스트리밍 출력 + 호출 구조 리팩터(`callGeminiStream/runGemini`)
4. 결과 저장/이전 기록(+JSON 내보내기/가져오기), .md/.doc 다운로드
5. 조건 프리셋
6. 후속 질문(멀티턴), 발표 슬라이드 개요
7. 예산 계산기(AI 단가 추정 + 직접 수정)
8. 모바일 레이아웃, 출처 표시 개선
9. 다크모드 + 보고서 표지(회사/로고/작성자) + 연관 키워드 추천
10. 토스 스타일 테마, ITCEN 로고/제목(VisionCenter 교육기획 AI Agent) 브랜딩
11. 품의서·교육 계획서·설문(사전/사후) 생성, 결과 공유 링크
12. 커리큘럼 세부 학습내용/목표 구체화(셀 줄바꿈), 표 글씨 축소
13. 첫 사용 가이드(온보딩)
14. 임원 예상질문&방어, 레드팀 허점 검토
15. 추가 문서 버튼 **열고닫기(토글)** + 닫힌 문서 내보내기 제외
16. 예산 입력칸 토스식 금액 박스로 재디자인(`.bfield`: 라벨+큰 숫자+단위)
17. 📈 ROI·기대효과 산출 카드 추가(예산 계산기 값 연동, AI 향상률 추정, ROI/순효과/회수기간 자동 계산, 생성 문서 '기대효과'에 반영)

---

## 11. 향후 개선 아이디어 (선택)

- **실제 키로 end-to-end 테스트**(가장 우선) — 스트리밍/문서생성/예산 AI추정/공유링크 열기 확인
- 안내·섭외 메일 초안, PWA 설치, 비용 구성 그래프, 비교 모드, 영문 보고서, 가상 수강생 페르소나, ROI 시뮬레이터
- 공용 키 + 작은 백엔드(키 1개 운영, 단 비용 집중)
- 다른 AI 제공자 지원(호출 함수 분리)

---

## 12. 막혔을 때 디버깅

- 결과 칸의 **"오류가 났어요: ..."** 문구로 원인 파악
  - `API key not valid` / `API_KEY_INVALID` → 키 문제(AIza 키 맞는지)
  - `quota` / `rate limit` → 무료 한도/혼잡 → 잠시 후 재시도
  - `model ... not found` → 모델명 변경 → `MODELS` 수정
  - 스트리밍 중단/네트워크 오류 → 다시 조사
- 브라우저 **F12 → Console / Network** 탭에서 상세 확인
- 코드 수정 후 배포 전, JS 문법 빠르게 점검:
  ```bash
  node -e "const fs=require('fs');const h=fs.readFileSync('index.html','utf8');new Function(h.match(/<script>([\s\S]*?)<\/script>/)[1]);console.log('OK');"
  ```

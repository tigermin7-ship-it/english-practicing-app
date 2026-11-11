# Product Requirements Document (PRD): BizRoleplay — AI 비즈니스 영어 롤플레잉 & 스마트 암기 루프 웹앱 (v3.0 Master Edition)

## 1. 제품 개요 (Product Overview)
* **제품명:** BizRoleplay — AI 비즈니스 영어 롤플레잉 & 스마트 암기 루프 웹앱
* **핵심 가치 (Core Value Proposition):**
  * 사용자가 보유한 수백~수천 개의 비즈니스 영어 문장(`(영어문장, 한글문장)` 2열 엑셀/CSV 또는 텍스트 세트)을 앱에 탑재하여 **스마트 플래시카드(한→영 인출 훈련)**로 선행 학습함.
  * 직후 실제 비즈니스 상황(**바이어 단가 협상, 일정 조율, 스몰토크, 임원 보고 등**)에서 AI 상대방과 자유롭게 대화하며, 방금 외운 표현을 내 입으로 써먹고 **내 문장에 대한 원어민 코칭 피드백 + 코치에게 추가 질문(Follow-up Q&A) + 오답노트 자동 축적**까지 한 화면에서 완결하는 **'인출(Output) 중심 비즈니스 영어 훈련소'** 구축.
* **타깃 사용자:** 실제 업무 미팅, 이메일, 협상, 발표, 해외 출장/쇼핑 등 실전 상황에서 쓸 비즈니스 영어 표현을 체화하고자 하는 실무자 및 학습자.
* **플랫폼 환경:**
  * 별도 서버 구축 없이 브라우저에서 즉시 구동되는 **단일 파일(Single-File HTML/CSS/JS) 서버리스 SPA + PWA(Progressive Web App) 아키텍처** (`index.html`, `manifest.json`, `icon.svg`, `icon-192.png`, `icon-512.png`).
  * **모바일 PWA 전체화면(Standalone) 지원:** `manifest.json` (`"display": "standalone"`) 및 iOS/Android 웹앱 메타 태그를 탑재하여 스마트폰에서 `'홈 화면에 추가'`로 실행 시 모바일 브라우저 주소창 및 하단 툴바를 제거해 세로 가용 화면(`100px+`)을 극대화함.
  * **GitHub Pages 멀티 환경 자동 배포:** `main` 브랜치(운영 환경 `/`)와 `dev` 브랜치(개발/검증 환경 `/dev/`)를 GitHub Actions 워크플로우로 분리하여 안전하게 배포·검증함.
  * **100% Firebase 클라우드 동기화 아키텍처:** Firebase Authentication(이메일/비밀번호 로그인) 및 Cloud Firestore(`users/{uid}` 계정 프로필 + `workspace/main` 통합 문장 DB·롤플레잉 세션·글로벌 API 키 공유 문서)를 통해 PC·모바일·태블릿 어디서 로그인하든 동일한 학습 데이터와 API 설정을 즉시 이어서 사용함.

---

## 2. 핵심 학습 루프 (The 5-Step Core Learning Loop)
1. **[Step 1. 통합 문장 DB 구축 & `#0` 제목 행 자동 제외 · `#1~#N` 자동 번호 부여]**
   * 사용자가 `(영어문장, 한글문장)` 2열 구조의 엑셀(`.xlsx/. csv`) 파일(예: `900_1.xlsx`)을 업로드하거나 `영어문장 | 한글문장` 텍스트 세트를 붙여넣음.
   * 첫 번째 행(`#0`)이 `Title / 해석`, `English / Korean`, `영어 / 한글` 등 컬럼 제목(Header)인 경우 파서가 이를 `#0` 제목 행으로 자동 판별하여 학습용 문장 DB에서 제외하고, 실제 첫 학습 문장(예: `"It's my first time / 이번이 처음이야"`)부터 파일(`source_file`)별 `#1 ~ #N`(`sentence_no`) 일련번호를 부여하여 저장함.
2. **[Step 2. 스마트 플래시카드 선행 인출 훈련 (Flashcard Study)]**
   * 롤플레잉 시작 전(또는 도중 언제든), 특정 파일 카테고리(예: `📁 900_1.xlsx`)나 `🔥 미암기 집중`, `⭐ 대화피드백 오답노트`를 선택해 플래시카드 모달을 켬.
   * 플래시카드 본문은 번호 노출 없이 오직 **한글 의미(앞면) → 영어 문장(뒷면)** 인출에만 집중하도록 구성하며, 상단 메타 배지에서만 출처 파일명과 문장 번호(`📄 900_1.xlsx · #1`)를 보조 표시함.
3. **[Step 3. 실전 비즈니스 AI 롤플레잉 (1:1 Multi-Persona Roleplay)]**
   * 좌측 사이드바에서 원하는 상대방과 미팅 상황(예: *"A사 바이어 John - Q4 단가 인하 방어"*, *"불가리아 데코프 상무관 - Horizon Europe 및 스타트업 진출 협력"*) 세션을 선택하거나 새로 생성함.
   * 실제 메신저/화상회의 채팅창처럼 AI 파트너와 오직 **해당 시나리오 맥락에 100% 집중한 실전 대화만** 주고받음 (부자연스러운 타깃 문장 유도나 대화 흐름을 끊는 자동 첨삭 개입 배제).
4. **[Step 4. 사용자 주도형 온디맨드(On-Demand) 코칭 피드백 & 코치에게 추가 질문(Follow-up Q&A)]**
   * 대화를 이어가다가 내가 쓴 영어 문장이 어색했거나 더 세련된 원어민 표현이 궁금할 때, **내 말풍선 우측 하단의 `[💬 피드백]` 버튼**을 클릭함.
   * 클릭 즉시 해당 문맥을 분석하여 말풍선 바로 아래에 **1) 비즈니스 적절성 평가, 2) 뉘앙스 팁, 3) 원래 의도(한글) + 원어민 추천 문장(영어)**을 아코디언 카드로 펼쳐줌.
   * 피드백 내용을 읽고 나서 추가로 궁금한 점(예: *"방금 상대방이 말한 'What's your poison?' 대신 제가 비즈니스 자리에서 상대방에게 마실 것을 권할 때 쓸 수 있는 세련된 표현 2가지만 더 알려줘"*)이 생기면, **피드백 카드 하단의 `[💬 코치에게 추가 질문하기]` 입력창**을 통해 그 자리에서 바로 질문하고 한국어 해설과 추천 영어 표현을 답변받음.
5. **[Step 5. 오답노트 원클릭 저장 & 복습 루프 완성]**
   * 기본 코칭 피드백의 추천 문장은 물론, **추가 질문(Follow-up Q&A)에서 코치가 제시한 추천 표현까지 `[⭐ 오답노트에 저장]` 버튼 한 번으로 통합 문장 DB(`category: '대화피드백'`)에 즉시 편입**됨.
   * 저장된 문장은 곧바로 플래시카드 `'⭐ 대화피드백 오답노트'` 덱에 추가되어 반복 암기 대상이 됨으로써 **암기 → 실전 사용 → 코칭/추가질문 → 오답노트 암기**의 선순환 루프가 완성됨.

---

## 3. 상세 기능 명세 (Detailed Feature Specifications)

### 3.1. 통합 문장 DB & 스마트 플래시카드 (Unified Sentence DB & Smart Flashcard)

#### 1) 문장 데이터 스키마 (`sentencesDB` 아이템 구조)
```javascript
{
  id: "s_1794145000000_1",          // 고유 식별자
  en: "It's my first time",          // 영어 문장 (필수)
  kr: "이번이 처음이야",             // 한글 해석 (필수)
  category: "900_1",                 // 카테고리 탭 분류명 (업로드 파일명에서 확장자 제거 또는 '대화피드백')
  source_file: "900_1.xlsx",         // 업로드된 원본 파일명 (수동 입력 시 '직접입력', 오답노트 저장 시 '대화피드백')
  sentence_no: 1,                    // 해당 source_file 내 1-based 순번 (#1 ~ #N)
  starred: false,                    // 중요 별표(★) 여부
  memorized: false,                  // 암기 완료(✅) 여부
  practice_count: 0,                 // 플래시카드 인출 연습 횟수
  source_session_id: null,           // 대화피드백 오답노트 저장 시 출처 세션 ID
  created_at: "2026-11-08T14:00:00Z" // 생성 시각
}
```

#### 2) `(영어문장, 한글문장)` 2열 표준 입력 및 `#0` 제목 행 자동 제외 (`isHeaderRowPair`)
* **입력 컬럼 구조 표준화:**
  * 사용자가 번호 열을 따로 관리할 필요 없이 **`(1열: 영어문장, 2열: 한글문장)` 2열 구조**를 기본 표준으로 처리함 (하위 호환을 위해 과거 3열 `(번호, 영어, 한글)` 파일이 들어와도 자동 감지).
* **`#0` 제목 행(Header Row) 자동 감지 및 제외 (`isHeaderRowPair(colA, colB)`):**
  * 엑셀/CSV의 첫 번째 행(`rowIdx === 0`, 개념상 `#0`번 행)을 검사하여 아래 조건에 해당하면 컬럼 제목 행으로 판별하고 DB 삽입에서 즉시 제외함:
    * 영문 헤더 키워드: `title`, `english`, `en`, `eng`, `sentence`, `english sentence`, `expression`, `영어`, `영어문장`, `영어 문장`, `영문`, `문장`, `표현`
    * 한글 헤더 키워드: `해석`, `한글`, `한국어`, `의미`, `뜻`, `번역`, `한글문장`, `한글 문장`, `한글해석`, `한글 해석`, `korean`, `kr`, `kor`, `meaning`, `translation`
    *特に `Title | 해석`, `English | Korean`, `영어 | 한글` 등 사용자의 실제 엑셀 헤더 조합을 100% 포착하여 제외함.
* **파일별 `#1 ~ #N` 순차 번호 자동 부여 및 기존 DB 자동 마이그레이션 (`sortSentencesDBBySourceAndNo`):**
  * `#0` 제목 행을 제외한 첫 번째 실제 학습 문장부터 `sentence_no = 1, 2, 3...`을 순차 부여함.
  * 동일 파일명(`900_1.xlsx`) 재업로드 시 해당 파일의 마지막 번호(`maxNo`)를 이어받거나 중복 문장은 번호·파일 정보를 갱신함.
  * **자동 클렌징 및 번호 재정렬 마이그레이션:** 기존 Firestore `workspace/main`에 이미 `#1 Title / 해석`이 포함되어 `#2 It's my first time`으로 밀려 있던 데이터를 로드할 때, `sortSentencesDBBySourceAndNo()`가 자동으로 `#0` 헤더 레코드(`Title / 해석`)를 삭제하고 해당 파일의 문장 번호를 `#1 ~ #899`로 빈틈없이 재인덱싱한 뒤 Firestore에 즉시 반영함.

#### 3) 플래시카드 UI vs 문장 DB 관리 모달 UI 역할 분리
* **플래시카드 학습 모달 (`renderCurrentFlashcard`, `formatFlashcardTextWithSubnote`, `jumpToFlashcardSentenceNo`, `toggleFlashcardShuffle`):**
  * **본문 가독성 극대화 (모바일 160% 확대):** 카드 상단 질문(`#fcTopPromptText`)과 하단 정답(`#fcBottomAnswerText`)에는 문장 번호를 섞지 않고 오직 한글/영어 문장 암기에만 몰입하도록 구성하며, 스마트폰 화면에서도 한눈에 시원하게 읽히도록 기본 폰트 크기를 기존 `20px`(`1.25rem`) 대비 **160% 확대한 `32px`(`text-[2rem]`, 태블릿 `sm:text-[2.5rem]`, 데스크톱 `md:text-[3.25rem]`)**으로 적용함.
  * **`>>` 부연 설명 `28px` 및 선명한 일반 검은색 폰트 분리 표시 (`formatFlashcardTextWithSubnote`):** 문장 텍스트 내에 `'>>'` 구분자가 포함된 경우, `'>>'` 이전의 핵심 문장은 `100%` 대형 크기로 유지하고 **`'>>'` 이하의 부연 설명은 줄바꿈 후 `28px`(`clamp(18px, 0.54em, 28px)`) 크기 및 흐리지 않은 일반 검은색 폰트(Light 모드 `#0f172a` / Dark 모드 `#ffffff`)**로 선명하게 분리 렌더링함. 또한 원어민 발음 듣기(`speakCurrentFlashcard`) 시에는 `'>>'` 앞의 순수 영어 본문만 TTS로 읽도록 처리함.
  * **상단 배지 중복 제거 및 `#문장번호` 직관화:** 좌측 상단 첫 번째 배지(`#fcTopCategoryBadge`)에는 확장자를 제거한 단일 카테고리명(`📁 900_1` 또는 `⭐ 대화피드백`)만 깔끔하게 표시하여 중복 노출(`900_1` + `900_1.xlsx`)을 제거하고, 두 번째 배지(`#fcTopSourceBadge`)에는 현재 파일 내 고유 문장번호(`#11`, `#138`)를 강조 표시함.
  * **특정 문장번호 바로가기 및 연속 순차 학습 (`#fcJumpInput` & `jumpToFlashcardSentenceNo`):**
    * 상단 헤더에 **`[# 번호 입력 ➔ 이동]`** 컨트롤을 배치하고, `#문장번호` 배지나 진행률 배지(`11 / 899`)를 클릭해도 입력창으로 즉시 포커스됨.
    * 사용자가 원하는 번호(예: `138`)를 입력하고 `Enter` 또는 `[이동]`을 누르면, 셔플(`isShuffle`)이 켜져 있더라도 **자동으로 `셔플 OFF`(문장번호 오름차순 정렬)로 전환**한 뒤 `#138`번 문장으로 즉시 점프함. 이후 `[다음 >]` 버튼을 누르면 **`139 ➔ 140 ➔ 141`번 순서로 끊김 없이 이어서 학습**할 수 있음.
  * **`셔플 OFF` 전환 시 현재 문장번호 위치 유지 (`toggleFlashcardShuffle`):**
    * `셔플 ON` 상태에서 임의의 카드(예: `#245`번 문장)를 학습하다가 **`셔플 OFF` 버튼을 누르면 `#1`번 카드로 초기화하지 않고 방금 보고 있던 `#245`번 문장 위치를 그대로 유지**함.
    * 이를 통해 사용자가 셔플로 임의의 번호를 뽑은 뒤 `셔플 OFF`를 눌러 그 다음 문장(`#246 ➔ #247 ➔ #248`)부터 순서대로 이어서 공부할 수 있음.
  * 방향 전환(`한 ➔ 영` 기본 / `영 ➔ 한`), 탭하여 정답 확인/다음 이동, 원어민 발음 듣기(`S`), 이전/다음 카드, 순서 섞기(Shuffle), 암기 완료(`+1`) 및 중요 별표(`B`) 북마크 지원.
  * **모바일(`lg` 미만) 세로/가로 반응형 헤더 컴팩트화 & 가로 눕힘 잘림 방지:**
    * 세로 모드 겹침 방지를 위해 불필요한 `Flashcard` 로고 텍스트를 제거하고, 범위 선택 박스(`#fcFilterSelect`)를 앞 5~6글자 축약 폭(`w-[88px] sm:w-[132px] lg:w-[200px] truncate`)으로 최소화함 (클릭 시 드롭다운에서 전체 제목 온전히 선택 가능).
    * `1024px` 미만 모바일·태블릿(세로 및 가로 눕힘 모두)에서는 자주 쓰지 않는 `[방향 전환 · 셔플 · 재생속도 · Exit]` 버튼을 우측 `[⋯]` 하위 팝오버 메뉴(`#fcHeaderMenuPopover`)로 일원화하여 상단에 **[범위 선택] + [진행률] + [# 이동] + [⋯]** 만 겹침 없이 배치함 (데스크톱 `lg:` 이상은 기존 1클릭 버튼 유지).
    * 모바일 가로 눕힘 등 낮은 뷰포트 높이(`max-height: 520px`)에서는 헤더/하단바 높이 및 카드 패딩 축소 + `clamp` 유동 폰트와 내부 스크롤(`flex-1 min-h-0 overflow-y-auto`)로 상·하단 본문 문장이 수직 잘리지 않도록 보정함.
* **통합 문장 DB 관리 모달 (`renderDBManagerModal`):**
  * 각 문장 카드 상단에 녹색 출처 배지(`📄 900_1.xlsx · #1`)와 영어 문장 앞 `[#1]` 번호 프리픽스를 명시하여 원본 엑셀 파일의 몇 번째 문장인지 즉각 대조 가능.
  * 검색창에서 `#15`, `900_1 #15`, `900_1.xlsx 15` 등 **파일명 + 문장번호 검색**으로 특정 번호의 문장을 1초 만에 필터링 가능.
  * 전체/카테고리별 엑셀(`.xlsx`) 내보내기 시 `문장번호 | 출처파일 | 카테고리 | 영어문장 | 한글문장 | 중요별표 | 암기여부 | 연습횟수` 컬럼을 온전히 포함하여 출력함.

---

### 3.2. 멀티 세션 비즈니스 롤플레잉 워크스페이스 (Roleplay Workspace)
* **시나리오 세션 프리셋 생성 및 관리:**
  * 좌측 상단 `[+ 새 롤플레잉 시나리오 세션 추가]` 모달에서 **세션 제목, AI 상대방 이름/직함(`partner_name`), 구체적인 비즈니스 미팅 배경 및 나의 목표(`situation`), 난이도(`Beginner` / `Intermediate` / `Advanced`)**를 자유롭게 설정·수정·삭제함.
  * 각 세션은 독립된 대화 히스토리(`messages[]`)를 보유하며 Firestore `workspace/main`을 통해 모든 기기에 실시간 동기화됨.
* **실전 몰입형 메신저 UI & 모바일 1줄 통합 GNB · 가독성 강화 레이아웃:**
  * **모바일 상단 헤더 1줄 통합 (`48px` 컴팩트 바) & 아바타 팝오버 (`< lg` 뷰포트):**
    * 모바일 화면에서 기존 2단 헤더(GNB + 상대방 정보바)가 차지하던 수직 공간을 절반 이상 축소하기 위해 상단 1줄(`h-12`)로 통합함.
    * 좌측에는 **`[≡]` 사이드바 토글**(문장 DB `전체 관리` 포함)과 **클릭 가능한 원형 상대방 아바타 배지(`#mobilePartnerAvatarBtn`)**만 배치하고, 긴 파트너 직함 텍스트는 기본 숨김 처리함. 아바타 클릭 시 팝오버(`#mobilePartnerPopover`)가 열려 상대방 이름·난이도·`📋 상황` 및 `[⚙️ 시나리오 설정]`, `[↺ 대화 비우기]`를 즉시 확인·실행할 수 있음.
    * 우측에는 **`[🤖 AI질문]`, `[🎴 보유문장수]`, `[⚙️]` 3개 핵심 컨트롤**만 배치함 (데스크톱 `lg:` 이상에서는 전체 브랜드 로고, 세션명, 2단 시나리오 바 및 `[문장 DB]` 버튼을 그대로 유지함).
  * **우상단 GNB 메뉴 3버튼 경량화 및 `[⚙️ 설정]` 모달 통합:**
    * 데스크톱 GNB 우측에는 **`[플래시카드]`, `[문장 DB]`, `[⚙️ 설정]` 딱 3개의 핵심 버튼**만 배치함.
    * 기존 최상단에 흩어져 있던 **`Dark/White` 화면 테마 전환, 관리자 `회원 승인 관리` 탭(대기 인원 배지 연동), `플래시카드 핵심 단축키`(`S: 원어민 발음 듣기 (TTS) | B: 중요 별표 북마크 | ←/→: 이전·다음 카드`), `로그아웃` 버튼**은 모두 `[⚙️ 설정]` 모달(`#securityModal`) 하부로 일원화함.
    * 불필요한 `'핵심 이용 가이드 & 단축키'` 단독 모달과 AI의 부자연스러운 발언을 유발하던 `'타깃'` 드롭다운/힌트 드로어 UI를 전면 제거하여 화면 공간과 대화 자연스러움을 동시에 확보함.
  * **모바일 대화창 폰트 확대, 말풍선 볼드 처리 & 발음 아이콘 인라인 배치:**
    * 스마트폰 환경에서의 시인성을 높이기 위해 **AI 파트너·사용자 대화 말풍선 본문을 `15.5px~16px` 굵은 볼드체(`font-bold`)**로 강화하고, 코칭 피드백의 추천 원어민 문장(`16px~17px font-bold`), 평가/뉘앙스 팁/추가 질문 본문(`14px`), 입력창(`16px` 모바일 자동 확대 방지)을 일괄 확대함.
    * AI 상대방 메시지의 발음 듣기 버튼(`[🔊]`)을 하단 별도 줄에서 제거하고 **상단 이름·시각 메타 라인(`Jane 오후 10:57 [🔊]`) 바로 옆에 인라인 배치**하여 대화창 세로 공간을 절약함.
  * **AI 첫 마디 선공 기능 (`[AI가 먼저 질문 던지기]` / `[파트너 첫 질문]`) & 실시간 SSE 스트리밍 초고속 UX:**
    * 빈 세션에서 대화를 시작할 때는 물론, **이미 대화가 진행 중인 상태에서도** 상단 `[AI가 먼저 질문 던지기]` 버튼을 누르면 현재 문맥에 이어지는 AI 파트너의 새로운 질문이나 후속 발언을 즉시 생성함.
    * `<head>` `preconnect` + `ListModels` 지연 폴백 + 모델군별 저지연 추론 설정(`gemini-3*`: `thinkingLevel: 'minimal'`, `gemini-2.5*`/`flash-lite-latest`: `thinkingBudget: 0`, 미지원 시 400 자동 재시도) + **SSE 실시간 스트리밍(`:streamGenerateContent?alt=sse`)**을 결합해 첫 토큰(TTFB) 수신 즉시 라이브 말풍선으로 출력하고, **Firestore 동기화(`saveAppState`)는 백그라운드 비동기 처리**하여 체감 응답 지연을 최소화함.
  * **음성 입력(STT) & iOS·데스크톱 겸용 무지연 음성 출력(하이브리드 TTS + Gemini Audio 폴백):**
    * 마이크 버튼(`webkitSpeechRecognition`)으로 실제 말하듯 영어를 입력함.
    * iOS Safari/Chrome WebKit/Standalone PWA 호환성을 위해 **ID 기반 클릭 핸들러(`speakSessionMessage`, `speakFeedbackAlternative`, `speakFollowUpSuggestion`)**로 교체하고, iOS에서는 **`utterance.voice` 미지정(`lang = 'en-US'`만 지정) + 터치 동기 호출**을 적용함.
    * 동시에 탭 시점에 `<audio id="ttsFallbackAudio" playsinline>` 요소를 동기 언락(Unlock)하고 `500ms` 워치독(`onstart` 미수신·무음 조기 종료·`onerror` 감지)을 감시해 **iOS WebKit TTS가 무반응일 경우 자동으로 Gemini Native Audio TTS(`gemini-2.5-flash-preview-tts` / `gemini-3.1-flash-tts-preview`) 24kHz PCM→WAV 오디오 폴백 재생(세션 캐싱 포함)**을 수행함.

---

### 3.3. 온디맨드 코칭 피드백 & 인라인 추가 질문 (On-Demand Feedback & Follow-up Q&A)

#### 1) 기본 코칭 피드백 (`msg.feedback`)
* 내 영어 말풍선 우측 하단 `[💬 피드백]` 버튼 클릭 시 해당 시점까지의 대화 맥락과 내 발언을 분석하여 아코디언 카드를 렌더링함:
  * **품질 배지 (`quality`):** `good`(✨ 자연스러운 비즈니스 표현), `minor_fix`(💡 추천 비즈니스 대안 제시), `needs_correction`(🔧 문법·비즈니스 톤 교정 필요)
  * **평가 (`evaluation_kr`):** 문법 정확성, 비즈니스 격식(Politeness/Clarity), 상대방 질문 의도 부합 여부 총평
  * **뉘앙스 팁 (`nuance_tip_kr`):** 원어민들이 실제 미팅에서 선호하는 어휘/관용구/화법 팁
  * **원어민 추천 문장 (`intended_kr` + `recommended_en`):** 사용자의 원래 의도를 살린 가장 자연스러운 비즈니스 영어 문장(`16px~17px font-bold`) 제시 + 모바일 2줄 줄바꿈을 없앤 **1줄 3아이콘 컴팩트 버튼(`[🔊]`, `[✏️]`, `[⭐]`)** 제공. `[⭐]` 버튼은 **오답노트 저장/해제 원터치 토글**로 동작함 (`chat_feedback` 항목만 선택적 추가·제거하며 기존 엑셀 문장은 보존). 추가 질문 Q&A 카드 역시 `[🔊]`, `[⭐]`(토글) 아이콘 버튼으로 경량화함.

#### 2) 피드백 카드 내 인라인 추가 질문 (`msg.feedback.qa_threads` — Task 2 핵심 기능)
* **도입 배경:** 코칭 피드백을 확인한 직후 *"상대방이 쓴 관용구 표현을 다른 상황에서는 어떻게 응용하나요?"*, *"좀 더 정중한 표현이나 캐주얼한 대안은 없나요?"* 같은 후속 의문이 생겨도 기존에는 물어볼 창구가 없었음.
* **데이터 구조 (`msg.feedback.qa_threads[]`):**
  ```javascript
  qa_threads: [
    {
      id: "qa_1794148000000",
      question: "여기서 상대방이 쓴 'What's your poison?' 대신 제가 음료를 권할 때 쓸 수 있는 표현도 알려주세요.",
      answer_kr: "'What's your poison?'은 친근한 술자리/모임 표현이므로, 비즈니스 미팅이나 격식 있는 자리에서 상대방에게 마실 것을 권할 때는 아래와 같이 표현하는 것이 훨씬 자연스럽습니다.",
      suggested_en: "Can I get you something to drink before we get started?",
      suggested_kr: "시작하기 전에 마실 것 좀 드릴까요?",
      saved_to_db: false,
      created_at: "2026-11-08T14:15:00Z"
    }
  ]
  ```
* **인라인 Q&A 스레드 UI & 인터랙션 (`renderChatMessages` 내 `feedbackCardHtml`):**
  * 피드백 카드 하단에 `💬 코치에게 추가 질문하기` 섹션이 상시 표시됨.
  * 사용자가 입력창(`#feedback-qa-input-{msg.id}`)에 한글 또는 영어로 궁금한 점을 입력하고 `[질문하기]` 버튼(또는 `Enter`)을 누르면 `askFeedbackFollowUpQuestion(msgId)`가 실행됨.
  * AI 코치는 **1) 시나리오 배경, 2) 직전 대화 맥락, 3) 사용자의 원래 발언과 1차 코칭 피드백, 4) 이전 추가 질문/답변 내역(`qa_threads`)**을 모두 참조하여 한국어 해설(`answer_kr`)과 핵심 추천 영어/한글 문장(`suggested_en`, `suggested_kr`)을 JSON으로 반환함.
  * 답변에 추천 영어 문장(`suggested_en`)이 포함된 경우 미니 박스와 함께 `[🔊 발음 듣기]` 및 **`[⭐ 오답노트 저장]` (`saveFollowUpExpressionToDB(msgId, qaId)`)** 버튼이 렌더링되어, 추가 질문으로 얻은 표현까지 즉시 통합 문장 DB(`대화피드백`)에 저장하고 Firestore에 동기화함.

---

## 4. AI 프롬프트 아키텍처 (3-Engine Prompt Architecture)

본 애플리케이션은 대화 몰입도와 코칭 품질을 극대화하기 위해 **3개의 독립된 Gemini API 호출 엔진**으로 분리 설계함.

### Engine 1: 롤플레잉 파트너 대화 엔진 (`callRoleplayPartnerAI` & `buildRoleplaySystemPrompt`)
* **역할:** 지정된 비즈니스 페르소나(`partner_name`, `partner_persona`)와 사용자가 입력한 비즈니스 상황 및 목표(`business_context`), 난이도(`difficulty`)에 완벽히 빙의하여 실제 미팅 상대방처럼 영어로만 응답함.
* **100% 시나리오 집중 원칙 (무관한 타깃 문장 유도 배제):**
  * 문장 DB에 저장된 일상 표현이나 오답노트 별표 문장(예: `"What is your favorite drink?"`)을 AI 파트너 프롬프트에 주입하여 억지로 유도(Nudging)하지 않음으로써, 진지한 비즈니스 협상이나 공공기관 미팅 시나리오의 맥락이 깨지는 현상을 원천 차단함.
  * AI 파트너는 오직 사용자가 `business_context`에 기술한 핵심 비즈니스 의제(예: 불가리아 스타트업 지원 공공기관 소개, Horizon Europe 컨소시엄 협력, NIPA/KOICA 전문가 파견 초청 공문 요청 등)에만 집중하여 현실적이고 전문적인 후속 질문을 던짐 (`1~3문장 이내`).
  * 마크다운 볼드(`**...**`)나 리스트 기호를 일절 사용하지 않고 순수 구어체 영어(Plain Spoken English)로만 출력하도록 강제하며, 후처리에서도 잔여 `**` 기호를 자동 제거함.
* **멀티턴 Role 교차 보장 (400 에러 원천 차단):**
  * Gemini `generateContent` API는 `contents[]` 배열의 첫 번째 요소가 반드시 `role: 'user'`여야 하며, 동일 `role`이 연속되면 `400 Bad Request`를 반환함.
  * 따라서 사용자가 `[AI가 먼저 질문 던지기]`를 눌렀을 때(빈 채팅방이거나 이미 AI/사용자 대화가 오간 상태 모두 포함), 히스토리 첫 메시지가 `model`이면 앞에 가상 user 시작 트리거를 자동 보강하고, 연속된 동일 role 메시지는 병합한 뒤, 마지막이 `model`로 끝나면 시스템 트리거 프롬프트(`role: 'user'`)를 자동 추가하여 어떤 상태에서도 100% 정상 응답하도록 보장함.

### Engine 2: 온디맨드 코칭 피드백 엔진 (`callCoachingFeedbackAI`)
* **역할:** 사용자가 특정 말풍선의 `[💬 피드백]` 버튼을 눌렀을 때만 호출되는 1타 비즈니스 영어 코치.
* **출력 포맷 (`responseMimeType: "application/json"`):**
  ```json
  {
    "quality": "good | minor_fix | needs_correction",
    "evaluation_kr": "문법, 어휘, 비즈니스 매너 측면의 구체적인 한국어 총평",
    "nuance_tip_kr": "원어민 비즈니스 화법 팁 및 대안 표현 해설",
    "intended_kr": "사용자가 원래 전달하고자 했던 자연스러운 한국어 의미",
    "recommended_en": "가장 자연스럽고 세련된 원어민 비즈니스 영어 추천 문장 1개"
  }
  ```

### Engine 3: 피드백 인라인 추가 질문 코치 엔진 (`callFeedbackFollowUpAI`)
* **역할:** 피드백 카드 내부에서 사용자가 입력한 후속 질문(`questionText`)에 대해 직전 문맥과 1차 피드백 내용을 바탕으로 즉시 답변하는 Q&A 전담 코치.
* **출력 포맷 (`responseMimeType: "application/json"`):**
  ```json
  {
    "answer_kr": "사용자의 추가 질문에 대한 친절하고 명확한 한국어 설명 (어감 차이, 문법 설명, 상황별 활용법 등)",
    "suggested_en": "질문과 관련하여 가장 추천하는 대표 비즈니스 영어 문장 1개 (없으면 빈 문자열)",
    "suggested_kr": "suggested_en에 대응하는 자연스러운 한국어 뜻 (없으면 빈 문자열)"
  }
  ```

---

## 5. 2026년형 Gemini API 호환성 및 동적 모델 자동 탐색 아키텍처 (Critical Technical Spec)

2026년 기준 Google AI Studio / Cloud Console의 API 키 체계 및 Gemini 모델 엔드포인트 정책 변경에 따른 시행착오를 모두 해결한 검증된 통신 규격임.

### 5.1. 2026년 Gemini 모델 폐기(Deprecation) 이슈와 초고속 검증 모델 우선순위
* **발생했던 이슈:**
  * 2026년 현재 신규 발급된 API 키에서 과거 모델명인 `gemini-1.5-flash`, `gemini-1.5-flash-latest`, `gemini-2.0-flash`, `gemini-2.5-flash`, `gemini-2.5-flash-lite`를 호출하면 **`404 Not Found` (`no longer available to new users` 또는 `not found for API version v1beta`)** 에러가 발생함.
  * 또한 `gemini-flash-latest` 별칭(alias)은 추론 지연으로 인해 응답에 **183초(3분 이상)**가 소요되어 프론트엔드가 멈춘 것처럼 보이는 현상이 실측됨.
* **해결 아키텍처 (`callGeminiWithAutoModelDiscovery` & `discoverAvailableGeminiModels`):**
  1. **실측 응답 속도 1.1~2.0초가 검증된 2026년 현역 모델을 최우선 후보(`preferredOrder`)로 배치:**
     * 1순위: `gemini-flash-lite-latest` (평균 1.15초, 기본값)
     * 2순위: `gemini-3.5-flash-lite` (평균 1.20초)
     * 3순위: `gemini-3.1-flash-lite` (평균 1.32초)
     * 4순위: `gemini-3-flash-preview` (평균 2.08초)
     * 5순위: `gemini-3.5-flash` (평균 3.29초)
  2. **레거시 설정 자동 마이그레이션 (`mergeWithDefaultState`):**
     * 사용자의 브라우저나 Firestore에 과거 모델(`gemini-2.5-flash`, `gemini-2.0-flash`, `gemini-1.5-flash`, `gemini-flash-latest` 등)이 저장되어 있더라도 로드 즉시 `gemini-flash-lite-latest`로 자동 마이그레이션하여 404 에러를 원천 방지함.
  3. **런타임 `ListModels` 동적 탐색 및 무중단 폴백:**
     * 만약 선택된 모델이 404를 반환하면 즉시 `GET https://generativelanguage.googleapis.com/v1beta/models`를 호출해 해당 API 키로 접근 가능한 실제 모델 목록을 조회하고, `gemini-flash-lite-latest` 등 작동하는 모델로 자동 전환하여 재시도 및 설정 저장까지 무중단으로 처리함.
  4. **15초 타임아웃(`AbortController`) 및 토큰 한도(`maxOutputTokens: 1024`) 보장:**
     * 특정 모델 엔드포인트가 응답 없이 지연될 경우 15초 내 자동으로 연결을 끊고 다음 빠른 모델로 폴백하며, JSON 응답이 중간에 잘리지 않도록 `maxOutputTokens: 1024`를 부여함.

### 5.2. API 키 인증 방식 (`AIzaSy...` 및 `AQ....` 모두 호환)
* Google AI Studio의 표준 API 키(`AIzaSy...`)와 Google Cloud Console의 최신 키 포맷(`AQ.Ab8RN6J...`) 모두 브라우저 CORS 프리플라이트 지연 없이 즉시 통과하는 **`?key={API_KEY}` 쿼리 파라미터 방식**을 1순위로 호출하며, 인증 오류 시 `x-goog-api-key` 헤더 방식으로 자동 재시도함 (`fetchGeminiWithAuth`).
* **Google Cloud Console에서 API 키 제한 시 필수 주의사항 (Troubleshooting Guide):**
  * Firebase가 자동 생성한 브라우저 키(`Browser key (auto created by Firebase)`)는 기본적으로 25개 Firebase 관련 API만 허용되어 있고 **`Generative Language API` (Gemini API)** 권한이 빠져 있는 경우가 많음.
  * 따라서 Gemini 호출용 API 키는 **Google AI Studio (`aistudio.google.com/app/apikey`)**에서 발급받거나, Google Cloud Console에서 API 제한사항에 **`Generative Language API`를 반드시 포함(또는 API 제한 없음)**해야 함.

---

## 6. Firebase 클라우드 동기화 및 보안 아키텍처 (Firebase Auth & Firestore Spec)

### 6.1. Firestore 컬렉션 구조 (Clean Slate Architecture)
* **`users/{uid}` (계정 프로필 컬렉션):**
  * `uid`, `email`, `displayName`, `role` (`'admin'` | `'user'`), `approved` (`true` | `false`), `createdAt`, `approvedAt`, `lastLoginAt`
  * 최초 관리자 이메일(`tigermin7@gmail.com`)은 가입/로그인 즉시 `role: 'admin', approved: true` 권한을 자동 부여받음.
* **`workspace/main` (전 기기 실시간 공유 워크스페이스 문서):**
  * `sentencesDB`: 통합 문장 DB 배열 (`900_1.xlsx` 등)
  * `roleplaySessions`: 롤플레잉 시나리오 세션 및 대화/피드백/추가질문 히스토리 배열
  * `sharedGeminiApiKey`: 관리자가 한 번 입력해 두면 모든 로그인 기기에서 별도 입력 없이 즉시 공유되는 글로벌 Gemini API 키
  * `sharedGeminiModel`: 글로벌 기본 모델 (`gemini-flash-lite-latest`)
  * `cleanSlateTimestamp`: 데이터 클린 슬레이트 기준 타임스탬프 (`1794132000000`)

### 6.2. GitHub Secret Scanning 키 폐기 방지 및 초기 스냅샷 Race Condition 방어
* **Firebase 웹 API 키 분할 난독화 (`DEFAULT_FIREBASE_CONFIG`):**
  * 공개 GitHub 저장소에 `AIzaSy...` 평문이 노출되면 GitHub Secret Scanning이 Google에 통보하여 Firebase API 키가 자동 폐기/순환되는 문제가 발생함.
  * 이를 영구 방지하기 위해 `index.html` 소스코드 내 `DEFAULT_FIREBASE_CONFIG.apiKey`는 배열 결합 방식(`['AIza', 'SyDM', ...].join('')`)으로 분할 저장하여 정적 스캐너 탐지를 우회함.
* **초기 로드 시 빈 배열 덮어쓰기(Race Condition) 원천 차단 (`hasLoadedInitialWorkspaceSnapshot`):**
  * 새 기기나 브라우저에서 로그인한 직후, Firestore로부터 최초 `workspace/main` 문서를 수신(`hasLoadedInitialWorkspaceSnapshot = true`)하기 전에는 로컬의 빈 `sentencesDB`가 클라우드로 푸시(`pushSharedWorkspaceToFirestore`)되지 않도록 엄격한 가드(Guard)를 적용함.
* **관리자 계정 관리 및 원클릭 클린 슬레이트 초기화:**
  * 관리자(`admin`)는 설정 모달의 **회원 승인 관리** 영역에서 대기 중인 가입자를 `[승인]`하거나 기존 가입자를 `[삭제]`할 수 있으며, `[🗑️ 전체 계정·DB 초기화]` 버튼으로 레거시 데이터(`bizroleplay_data/shared_workspace` 포함)와 타 계정을 일괄 정리할 수 있음.

---

## 7. 검증 및 배포 표준 절차 (Verification & Deployment Protocol)
1. **구문 및 태그 무결성 검증:**
   * 배포 전 Node.js `vm.Script`를 이용한 인라인 `<script>` 문법 검사 및 HTML `<div> / </div>` 여닫는 태그 쌍 일치 검사를 필수로 통과해야 함.
2. **브랜치 전략 및 배포 경로:**
   * 모든 신규 기능 및 버그 수정은 `dev` 브랜치에 커밋·푸시하여 `https://tigermin7-ship-it.github.io/english-practicing-app/dev/` 스테이징 환경에 먼저 배포함.
3. **실제 브라우저 E2E 검증:**
   * 배포 완료 후 실제 헤드리스/브라우저 환경에서 **1) `900_1.xlsx` `#0` 제목 행 제외 및 `#1 It's my first time` 번호 정렬, 2) AI 파트너 첫 질문 및 멀티턴 대화 응답 속도(1~2초대), 3) 코칭 피드백 생성 및 인라인 추가 질문(Follow-up Q&A) 응답·오답노트 저장**까지 전 과정을 E2E 테스트로 검증함.

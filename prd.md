# [통합 마스터 PRD] 비즈니스 영어 실전 롤플레잉 & 플래시카드 올인원 웹앱 (BizRoleplay)

> **문서 목적:** 본 문서는 초기 기획서(`prd.md`)와 추가 업그레이드 요구사항(`prd_v2_upgrade.md`)을 하나로 통합한 **유일한 공식 마스터 기획서(Single Source of Truth)**입니다. 향후 모든 기능 개발, 수정 및 소통은 본 `prd.md`를 기준으로 진행합니다.

---

## 0. 핵심 설계 원칙 및 아키텍처 대전제 (Core Mandates)

1. **100% Firebase / Firestore 클라우드 단일 아키텍처 (로컬 인증·로컬 전용 저장 전면 배제):**
   - 브라우저 로컬 저장소(`LocalStorage`)에 로그인 정보를 담거나 `localhost` 전용 로컬 폴백(Fallback)으로 테스트하는 구성을 **전면 삭제**함.
   - **모든 로그인 및 회원 권한 관리**는 오직 **Firebase Authentication**과 **Cloud Firestore(`users` 컬렉션)**를 통해서만 처리함.
   - **모든 학습 데이터(문장 DB, 롤플레잉 세션, 유료 API 키, 테마 설정)** 역시 브라우저 임시 저장소가 아닌 **Cloud Firestore(`bizroleplay_data/shared_workspace`)**에 실시간 저장·동기화하여, PC·스마트폰·태블릿·다른 브라우저 어디서 접속하더라도 동일한 데이터가 즉시 노출되도록 함.
2. **에이전트 대화형 설정 수집 프로토콜 (Grill-Me 방식):**
   - 개발·배포·유지보수 과정에서 **Firebase/Firestore 설정값(`firebaseConfig`), Firestore 보안 규칙, 관리자 계정 정보, 유료 Gemini/OpenAI API Key** 등 사용자 고유 설정이 필요한 경우, 에이전트가 임의로 로컬 임시방편(Mock/Fallback) 코드를 작성하지 않음.
   - 반드시 **에이전트가 사용자에게 대화형(Grill-Me 방식)으로 필요한 정보를 하나씩 물어보고**, 사용자의 답변을 받아 코드 상수 또는 클라우드 설정에 직접 반영함.
3. **모바일 화면 최적화를 위한 미니멀 UI & 팝업 도움말(`[❓]`):**
   - 작은 스마트폰 화면을 복잡하게 만드는 상시 안내문(`"머릿속으로 영작하거나 입 밖으로 말해본 뒤..."`, `"먼저 스스로 문장을 인출(Retrieval)해 봐야..."`, `"STEP 1. 제시된 한국어 의미를 보고..."` 등)은 화면에서 **전면 제거**함.
   - 이용법 및 단축키 안내는 상단 헤더의 **`[❓]` 버튼 클릭 시 열리는 팝업 모달(`#helpGuideModal`)**로만 깔끔하게 제공함.
4. **유료 API 기반 실전 AI 롤플레잉 & 온디맨드 피드백:**
   - 사용자의 유료 Google Gemini API(`gemini-2.5-flash` / `gemini-2.5-pro`) 또는 OpenAI API 키를 Firestore 설정에 연동하여, 시뮬레이션이 아닌 **실제 원어민 비즈니스 파트너 수준의 실시간 대화 및 정밀 교정 코칭**이 전 기기에서 동일하게 작동하도록 함.

---

## 1. 프로젝트 개요 및 비전 (Project Overview)

- **제품명:** BizRoleplay (비즈니스 영어 실전 체화 및 인출 훈련 플랫폼)
- **개발 및 배포 형태:**
  - 단일 파일 웹 애플리케이션 (`index.html` 내 HTML5, Tailwind CSS CDN, Lucide Icons CDN, SheetJS CDN, Firebase SDK CDN, Vanilla JS 통합)
  - GitHub Pages 배포 (안정 운영 버전 `main` 브랜치 `/` 및 테스트 버전 `dev` 브랜치 `/dev/` 이원화 운영)
- **핵심 가치:**
  - 외국인과의 접점이 없는 일상에서도 **'영어로 생각하고 즉각 문장화하는 환경'**을 메신저 대화 형태로 일상화.
  - **사전 인출 훈련(플래시카드) ➔ 실전 적용(무중단 롤플레잉) ➔ 선택적 교정(인라인 댓글 피드백) ➔ 오답 축적(Firestore 통합 DB 아카이빙)**의 순환 학습 루프 완성.

---

## 2. 문제 해결 매트릭스 (Context & Solutions)

| 사용자의 실제 페인포인트 | BizRoleplay의 기술적·UX적 해결 방안 |
| :--- | :--- |
| **1. 아무나 접속 가능한 보안 문제** | **Firebase Auth + Firestore 관리자 승인제 로그인:** ID(이메일)/PW 입력 후 Firestore에서 관리자가 승인(`status: "approved"`)한 계정만 앱 진입 허용 |
| **2. 기기·브라우저 변경 시 업로드한 문장 증발** | **Cloud Firestore 공통 DB 실시간 연동:** 엑셀/CSV로 업로드한 문장, 오답노트, 세션 내역, API 키를 Firestore에 통합 저장하여 모든 기기에서 실시간 공유 |
| **3. 휴대폰 화면을 가리는 복잡한 이용 설명문** | **미니멀 UI & `[❓]` 도움말 팝업화:** 매 화면 반복되던 STEP 안내문과 코칭 예시문을 전면 삭제하고 상단 `[❓]` 팝업 모달로 분리 |
| **4. 일상 속 영어 사고의 비지속성** | **AI 선제 발화(Proactive Prompting) & 다중 세션:** AI 파트너가 비즈니스 상황을 가정해 먼저 질문과 요청을 던져 자연스럽게 영어 답변을 유도 |
| **5. 기존 AI 챗봇의 역할 이탈 및 대화 단절** | **이원화된 프롬프트 엔진(Dual-Prompt Engine):** 롤플레잉 대화 생성 API와 인라인 피드백 API를 물리적으로 분리하여 대화 중 문법 훈수를 100% 금지 |
| **6. 사전 문장 암기 및 영작 인출 연습 필요** | **전체 화면 2단계 분할 플래시카드 모달:** 상단 50%에 한글 노출 ➔ 클릭 시 하단 50%에 영문 정답 노출. `[✕ Exit]` 클릭 시 원래 대화창으로 즉시 복귀 |

---

## 3. 시스템 아키텍처 및 기술 규격 (Technical Specifications)

- **Single-File 구성:** 모든 마크업, 스타일, 비즈니스 로직을 단일 `index.html` 파일로 구현.
- **프론트엔드 및 클라우드 스택:**
  - **UI/Styling:** Tailwind CSS CDN (`https://cdn.tailwindcss.com`) + Dark / White 테마 전환 지원
  - **Icons:** Lucide Icons CDN (`https://unpkg.com/lucide@latest`)
  - **Excel/CSV Parser:** SheetJS CDN (`https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js`)
  - **Authentication & Database:** Firebase v10 Compat CDN (`firebase-app-compat.js`, `firebase-auth-compat.js`, `firebase-firestore-compat.js`)
- **기본 내장 Firebase 프로젝트 구성 (`DEFAULT_FIREBASE_CONFIG`):**
  - 접속할 때마다 설정값을 수동 입력할 필요 없이 코드 상단에 `DEFAULT_FIREBASE_CONFIG`(`projectId: "english-practicing-app"`)를 내장하여 앱 실행 즉시 Firebase Auth 및 Cloud Firestore와 자동 연결됨.
- **LLM API 연동 규격:**
  - 지원 엔진: 유료/무료 Google Gemini API (`gemini-2.5-flash`, `gemini-2.5-pro`, `gemini-2.0-flash`) 및 OpenAI API (`gpt-4o-mini`, `gpt-4o`).
  - 피드백 엔진 호출 시 `responseMimeType: "application/json"` 옵션을 사용하여 파싱 오류 없는 정형 JSON 응답 보장.
  - 등록된 API 키는 Firestore `workspace/main` 내 `settings`에 저장되어 다른 기기에서 로그인 시 재입력 없이 즉시 활성화됨.

---

## 4. Cloud Firestore 데이터 스키마 설계 (Data Models)

모든 데이터는 로컬 브라우저 종속 없이 Cloud Firestore에 저장·동기화됩니다.

### 4.1 사용자 및 승인 권한 컬렉션 (`Firestore: users/{uid}`)
Firebase Authentication으로 가입/로그인한 사용자의 승인 상태와 권한을 관리합니다.
- 단순 ID(예: `admin`) 입력 시 내부적으로 `admin@bizroleplay.local` 이메일 포맷으로 자동 변환하여 Firebase Email/Password 인증을 수행합니다.
- `users` 컬렉션이 비어 있을 때 최초 가입하는 1번째 계정은 자동으로 관리자(`role: "admin"`, `status: "approved"`)로 지정되며, 이후 가입자는 `role: "user"`, `status: "pending"` 상태로 생성되어 관리자의 승인을 기다립니다.

```json
{
  "uid": "firebase_auth_uid_123",
  "email": "admin@bizroleplay.local",
  "displayId": "admin",
  "role": "admin",
  "status": "approved",
  "created_at": 1730990000000
}
```

### 4.2 전 기기 공유 워크스페이스 문서 (`Firestore: workspace/main`)
승인된 사용자가 어떤 기기에서 접속하든 동일한 학습 환경을 제공하는 통합 문서입니다.
- `sentencesDB`: 다중 엑셀/CSV 업로드 문장, 텍스트 붙여넣기 문장, 대화 중 저장한 오답노트 문장 배열
- `sessionsDB`: 다중 비즈니스 롤플레잉 세션 및 대화 기록 배열
- `activeSessionId`: 마지막으로 활성화된 세션 ID
- `settings`: AI 공급자(`provider`), 모델(`model`), 유료 `apiKey`, TTS 속도(`ttsSpeed`), 화면 테마(`theme: 'dark' | 'light'`)

#### (1) `sentencesDB` 항목 구조
```json
[
  {
    "id": "sent_1729800001_a1b",
    "source_file": "해외바이어_단가협상.xlsx",
    "category": "단가협상",
    "korean": "주문 물량을 늘려주신다면 5% 추가 할인을 검토해 보겠습니다.",
    "english": "If you increase the order volume, we can consider an additional 5% discount.",
    "notes": "조건부 양보 협상 핵심 표현",
    "practice_count": 2,
    "is_starred": true,
    "created_at": 1729800001000
  },
  {
    "id": "sent_1729800500_x9z",
    "source_file": "chat_feedback",
    "category": "대화피드백",
    "korean": "현재 내부 승인 절차를 밟고 있으며 금주 내로 확정안을 공유드리겠습니다.",
    "english": "We are currently going through internal approval and will share the finalized plan by the end of this week.",
    "notes": "[원래 내 문장] We are checking inside and will tell you this week. -> 비즈니스 격식체로 교정",
    "practice_count": 0,
    "is_starred": true,
    "created_at": 1729800500000
  }
]
```

#### (2) `sessionsDB` 항목 구조
```json
[
  {
    "session_id": "sess_001",
    "title": "A사 바이어 John - Q4 단가 인하 방어",
    "partner_name": "John Miller (Purchasing Director)",
    "partner_persona": "비용 절감 압박을 받고 있어 단가 10% 인하를 강하게 요구하는 노련한 해외 바이어",
    "business_context": "연간 재계약 협상 미팅. 당사는 원자재 가격 상승으로 단가 인하가 불가능하며, 대신 물량 확대 시 5% 할인을 제안해야 함.",
    "difficulty": "Intermediate",
    "target_filter": "해외바이어_단가협상.xlsx",
    "updated_at": 1729801000000,
    "messages": [
      {
        "msg_id": "m_01",
        "role": "partner",
        "text": "Thanks for joining today. Being completely transparent, we need a 10% unit price reduction for Q4 to keep you as our primary vendor. How much room do you have here?",
        "timestamp": 1729800900000
      },
      {
        "msg_id": "m_02",
        "role": "user",
        "text": "10% is too hard for us because material cost went up. But if you order more, we can give 5% discount.",
        "timestamp": 1729800950000,
        "feedback": {
          "is_open": true,
          "target_used": true,
          "matched_sentence_id": "sent_1729800001_a1b",
          "evaluation": "의미는 명확히 전달되었으나, 임원급 협상에서는 'too hard'나 'give discount'보다 격식 있는 비즈니스 표현이 유리합니다.",
          "recommended_korean": "원자재 가격 상승으로 10% 인하는 어렵습니다만, 주문 물량을 늘려주신다면 5% 추가 할인을 검토해 보겠습니다.",
          "better_alternative": "A 10% reduction is challenging due to rising raw material costs. However, if you increase the order volume, we can consider an additional 5% discount.",
          "nuance_tip": "'too hard' 대신 'challenging'을, 'give discount' 대신 'consider an additional discount'를 사용하면 협상 주도권을 유지할 수 있습니다.",
          "saved_to_db": true
        }
      }
    ]
  }
]
```

---

## 5. 핵심 기능 상세 요구사항 (Functional Requirements)

### 5.1 Firebase 로그인 게이트 & 관리자 회원 승인 관리
- **전면 로그인 게이트 모달 (`#authGateModal`):**
  - 비로그인 상태에서 앱 접속 시 전체 화면 로그인 모달을 최우선 표시하여 미승인 사용자의 무단 접근을 차단.
  - 상단 GNB 헤더에도 미인증 시 `[로그인]` 버튼(`#btnLoginHeader`), 인증 완료 시 `[로그아웃]` 버튼(`#btnLogoutHeader`)을 명시적으로 제공.
  - 일반 접속자에게 불필요한 설정 코드 입력창은 노출하지 않으며, 내장된 `DEFAULT_FIREBASE_CONFIG`로 즉시 Firebase Auth 및 Firestore에 연결.
- **승인 대기 화면 (`#authPendingView`) 및 관리자 승인 모달 (`#adminApprovalModal`):**
  - `status: "pending"` 또는 `"rejected"` 계정이 로그인하면 앱 진입을 차단하고 `"⏳ 관리자 승인 대기 중입니다"` 안내 화면 노출.
  - 관리자(`role: "admin"`)로 로그인하면 상단 GNB에 `[승인관리]` 버튼(`#btnAdminApprovalModal`)과 대기 인원 배지(`#gnbPendingUsersBadge`)가 활성화되며, 클릭 시 가입 신청자 목록을 조회하여 원클릭으로 `[✅ 가입 승인]` 또는 권한 변경 처리 가능.

### 5.2 다중 세션 프리셋 관리 및 시나리오 설정
- **세션 목록 및 간헐적 이어하기:**
  - 좌측 사이드바(모바일은 드로어 메뉴)에서 여러 개의 비즈니스 세션을 생성, 선택, 수정, 삭제 가능.
  - 세션을 클릭하면 이전 대화 내역(`messages[]`)과 설정값(페르소나, 비즈니스 상황, 난이도, 연동된 표현 필터)이 Firestore로부터 복원되어 어느 기기에서든 대화를 이어갈 수 있음.
- **난이도 조절(Difficulty Selector):** `Beginner (초급)`, `Intermediate (중급)`, `Advanced (고급)` 선택 가능.
- **현재 대화 비우기 vs 새 세션 추가 분리:** 시나리오 설정은 유지한 채 대화 기록만 초기화하는 `[🔄 대화 비우기]`와 신규 시나리오를 만드는 `[➕ 새 세션]`을 분리 제공.

### 5.3 무중단 메신저형 롤플레잉 대화창 (모바일 미니멀 UI 적용)
- **AI 선제 발화 기능 (`[🎲 파트너 첫 질문]`):**
  - 대화 내역이 비어 있는 세션에서 클릭 시, AI가 현재 시나리오와 타깃 표현(미학습·중요 문장 5~10개 스마트 샘플링)을 바탕으로 먼저 첫인사와 함께 비즈니스 질문을 던짐.
- **역할 엄격 고정 (Zero-Explanation Policy):**
  - 롤플레잉 대화 중 AI는 절대 문법을 지적하거나 제3자 시점에서 해설하지 않으며, 100% 비즈니스 파트너로서만 응답함.
- **모바일 화면 간소화:**
  - 채팅 입력창 하단의 고정 설명 문구와 상단 시뮬레이션 경고 배너를 제거하여 작은 스마트폰 화면에서도 채팅 메시지와 입력창만 깔끔하게 노출.
- **타깃 표현 힌트 서랍 (`[💡 힌트]`) 및 음성 입출력 (STT / TTS):**
  - 필요할 때만 열어보는 토글형 힌트 서랍 제공.
  - Web Speech API 기반 영어 음성 입력(`🎙️ STT`) 및 원어민 발음 듣기(`🔊 TTS`, 배속 `0.8x ~ 1.2x`) 지원.

### 5.4 온디맨드 인라인 댓글 피드백 & 원클릭 오답노트 저장
- **선택적 피드백 요청 (`[💬 피드백]`):**
  - 내 말풍선 아래의 `[💬 피드백]` 버튼을 눌렀을 때만 해당 말풍선 바로 아래에 접이식(Accordion) 댓글 카드가 열리며 유료 LLM API 코칭 결과 표시.
  - 이미 생성된 피드백은 재클릭 시 API 재호출 없이 즉시 열기/닫기 토글(`🔄 재분석` 버튼 별도 제공).
- **원클릭 통합 DB 저장 (`[⭐ 오답노트에 저장]`):**
  - 피드백 댓글 내 추천 한국어 의미 + 추천 원어민 영어 문장 쌍을 인라인 수정하거나 즉시 통합 문장 DB(`sentencesDB`, `category: "대화피드백"`)에 추가하고 Firestore에 실시간 반영.

### 5.5 스마트 문장 세트 입력 (엑셀/CSV 업로드 + 텍스트 붙여넣기) & Firestore 실시간 공유
- **다중 엑셀/CSV 업로드 및 자동 컬럼 매핑:**
  - `.xlsx`, `.xls`, `.csv` 파일을 SheetJS로 파싱하여 한글/영어 열을 자동 판별(`UTF-8` 및 `EUC-KR` 폴백 지원).
- **스마트 텍스트 일괄 붙여넣기 (Bulk Text Paste):**
  - 구분자(`탭`, `|`, `-`, `:`) 또는 한·영 줄바꿈 교차 텍스트를 붙여넣어 즉시 문장 세트로 등록.
- **Firestore 실시간 동기화:**
  - 업로드하거나 수정·삭제한 문장 세트는 즉시 Firestore `workspace/main`에 동기화되어, 다른 PC나 스마트폰에서 접속해도 별도 업로드 없이 동일한 문장 DB가 즉시 표시됨.
  - 전체 또는 필터별 문장 목록을 언제든 `.xlsx` 엑셀 파일로 다운로드 가능.

### 5.6 전체 화면 2단계 분할 플래시카드 모드 (군더더기 없는 모바일 인출 레이아웃)
- **독립 모달 진입 및 복귀:**
  - 상단 `[🎴 플래시카드]` 클릭 시 전체 화면 오버레이로 열리며, `[✕ Exit]`(또는 `ESC`) 클릭 시 이전 대화 스크롤 위치로 즉시 복귀.
- **불필요한 설명 문구 제거 (미니멀 50:50 분할):**
  - 상단 50% 영역에는 카테고리/연습 횟수 미니 배지와 **중앙의 제시문(한글 또는 영어)**만 큼직하게 노출(`"STEP 1..."`, `"머릿속으로 영작하거나..."` 등 장문 설명 제거).
  - 하단 50% 영역은 대기 상태에서 간결한 **`탭하여 정답 보기`** 아이콘만 표시하고, 화면 탭(또는 `Space`) 시 **영어 정답 문장, 메모, 액션 버튼(`🔊 발음`, `✅ 암기 완료 (+1)`, `★ 별표`)**만 깔끔하게 노출.
- **이용 가이드 분리 (`[❓]` 팝업 모달):**
  - 단축키(`Space/Enter`, `←/→`, `S`, `B`) 및 앱 사용법은 상단 GNB의 `[❓]` 버튼을 눌렀을 때만 팝업(`#helpGuideModal`)으로 안내.

---

## 6. 이원화된 AI 프롬프트 엔지니어링 명세 (Dual-Prompt Engine Specs)

### 6.1 엔진 1: 롤플레잉 파트너 전용 프롬프트 (Roleplay Engine)
> **목적:** AI가 설명 모드로 빠지는 것을 원천 차단하고, 100% 비즈니스 파트너 페르소나만 유지합니다.
```text
You are strictly roleplaying as a real-world business partner in a live messenger/meeting conversation.

[PARTNER PROFILE & SCENARIO]
- Your Name & Role: {partner_name} ({partner_persona})
- Business Situation & Context: {business_context}
- Difficulty & Tone Level: {difficulty} (Beginner: supportive & clear / Intermediate: realistic professional / Advanced: demanding, sharp, high-stakes negotiation)
- Target Expressions User Wants to Practice:
{sampled_target_expressions_list}

[ABSOLUTE RULES - VIOLATION WILL BREAK THE APP]
1. STAY IN CHARACTER 100%: Never act as an AI assistant, English teacher, or coach.
2. ZERO CORRECTIONS OR EXPLANATIONS: Never correct the user's grammar, never praise their English, and never explain the situation from a third-person perspective.
3. PROACTIVE & NATURAL DIALOGUE: Respond naturally in 1 to 3 concise business English sentences, then ALWAYS end with a realistic follow-up question, counter-offer, or business request that naturally prompts the user to reply (ideally nudging them to use one of the Target Expressions).
4. IF STARTING THE CONVERSATION: Greet the user briefly in character and immediately bring up the core business issue or question based on the scenario and target expressions.
```

### 6.2 엔진 2: 온디맨드 댓글 피드백 전용 프롬프트 (Feedback Engine)
> **목적:** 사용자가 `[💬 피드백]` 버튼을 눌렀을 때만 별도 호출되며, 정형화된 JSON만 반환합니다.
```text
You are an expert Executive Business English Coach. Analyze the user's English response within the given business conversation context and return ONLY a valid JSON object.

[CONTEXT]
- Business Scenario: {business_context}
- Partner's Previous Message: "{previous_partner_message}"
- User's English Message to Evaluate: "{target_user_message}"
- Registered Target Expressions for this Session: {sampled_target_expressions_list}

[OUTPUT JSON SCHEMA]
{
  "target_used": boolean,
  "matched_sentence_id": "string or null",
  "evaluation": "한국어로 작성된 1~2문장의 핵심 피드백 (비즈니스 격식, 문법 정확성, 뉘앙스 적절성 평가)",
  "recommended_korean": "추천 영어 대안 문장에 정확히 대응하는 자연스러운 비즈니스 한국어 문장 (오답노트 플래시카드 상단 노출용)",
  "better_alternative": "Polished, natural, native-level business English sentence that best fits the user's intent in this context (오답노트 플래시카드 하단 노출용)",
  "nuance_tip": "왜 이 대안 표현이 비즈니스 상황에서 더 효과적인지 설명하는 짧은 팁 (한국어)"
}
```

---

## 7. 에이전트 협업 및 유지보수 지침 (Antigravity Workflow Rules)

1. **로컬 전용/더미 코드 작성 금지:**
   - 모든 인증과 데이터 저장은 실제 Firebase Authentication 및 Cloud Firestore(`english-practicing-app`)를 기준으로 동작해야 하며, 로컬 브라우저 전용 로그인이나 로컬 전용 스토리지 코드로 대체하지 말 것.
2. **Grill-Me 방식 질문 및 설정 반영:**
   - Firebase/Firestore 추가 설정, 보안 규칙, 또는 유료 Gemini/OpenAI API 키 등 사용자 정보가 필요한 경우, 에이전트가 먼저 사용자에게 명확한 질문(Grill-Me)을 던져 값을 전달받은 뒤 설정을 완료할 것.
3. **사전 영향도 평가 및 구문 무결성 검증:**
   - 기존 핵심 기능(GNB, Firebase 로그인, 모달 시스템, Firestore 동기화 등) 수정 전 영향도 평가를 수행하고, 배포 전 HTML 태그 쌍 일치 및 JS 문법 검증(`vm.Script`)을 철저히 이행할 것.

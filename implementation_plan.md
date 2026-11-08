# 📋 통합 구현 계획서 (Implementation Plan)

본 계획서는 아래 **2가지 핵심 과제** 및 클라우드·API 안정화 사항을 통합 설계·구현한 문서임.
1. **과제 1:** 통합 문장 DB `(영어문장, 한글문장)` 2열 표준화, `#0` 제목 행(`Title / 해석`) 자동 감지·제외 및 `#1~#N` 순번 자동 정리
2. **과제 2:** 인라인 코칭 피드백 카드 내 **'코치에게 추가 질문하기 (Follow-up Q&A)'** 대화형 질의응답 기능 추가

---

## 📌 과제 1. 통합 문장 DB `(영어문장, 한글문장)` 2열 표준화 및 `#1~#N` 자동 인덱싱

### 1-1. 현황 진단 (`900_1.xlsx` 번호 밀림 원인)
* 업로드된 `900_1.xlsx`의 1행(`Title` | `해석`)은 열 제목(Header)이나, 기존 파서의 영문 헤더 키워드에 `'title'`이 없어 1행을 일반 학습 문장 `#1`로 등록함.
* 이로 인해 실제 1번 문장(`"It's my first time / 이번이 처음이야"`)이 `#2`로 밀리고, 플래시카드 및 AI 타깃 문장에 `"해석 ➔ Title"`이 포함되는 현상이 발생함.

### 1-2. 세부 구현 방안
1. **`(영어문장, 한글문장)` 2열 표준 파서 및 `#0` 제목 행 자동 제외 (`isHeaderRowPair`, `extractSentencesFromSheetRows`, `parseBulkSentencesText`)**
   * 기본 입력 구조를 **1열: `영어문장`, 2열: `한글문장`** (선택 3열: `메모`)으로 표준화함(기존 3열 번호 포함 파일도 하위 호환 유지).
   * 1행(`#0`행)이 `Title / 해석`, `English / Korean`, `영어 / 한글`, `Sentence / 뜻` 등 열 제목인 경우 `isHeaderRowPair(colA, colB)`를 통해 이를 **`#0` 헤더로 판별하여 학습 문장 배열에서 자동 제외**함.
   * 헤더 다음의 실제 첫 번째 문장(`"It's my first time"`)부터 파일명(`source_file`)별로 **`#1, #2, ..., #899`** 순번(`sentence_no`)을 자동 부여함.
2. **기존 업로드된 `900_1.xlsx` 데이터 자동 보정 (`sortSentencesDBBySourceAndNo`)**
   * 이미 Firestore(`workspace/main`)에 저장된 데이터 중 `Title / 해석` 헤더 항목을 자동 감지·제거하고, 각 파일(`900_1.xlsx`) 내 문장 번호를 **`#1 ~ #899`로 즉시 재정렬(Auto-Migration)**하여 재업로드 없이도 `"It's my first time"`이 `#1`로 표시되도록 처리함.
   * 자동 보정이 발생하면 즉시 Firestore `workspace/main`에도 클린 상태를 반영함.
3. **플래시카드 및 DB 관리 화면 연동**
   * 플래시카드 본문은 **순수 `영어-한글` 문장만 표시**하고, DB 관리 모달에서는 `📄 900_1.xlsx · #1` 배지 및 번호 검색(`1`, `#1`)으로 즉시 조회할 수 있도록 유지함.

---

## 📌 과제 2. 코칭 피드백 카드 내 '코치에게 추가 질문하기 (Follow-up Q&A)' 기능 추가

### 2-1. 필요성 및 사용자 시나리오
* 사용자의 발언에 대해 AI 코치가 `평가`, `뉘앙스 팁`, `추천 문장`을 제시했을 때(예: `'What's your poison?'` 관련 피드백), 사용자가 피드백 카드 안에서 즉시 추가 질문을 던지고 답변을 받을 수 있는 인터페이스를 제공함.
  * *질문 예시 1:* `"술을 안 마신다고 정중히 거절할 때는 영어로 어떻게 대답하면 좋을까?"`
  * *질문 예시 2:* `"'What's your poison?' 말고 비슷한 상황에서 원어민이 자주 쓰는 다른 표현도 알려줘."`
  * *질문 예시 3:* `"추천해 준 문장에서 'favorite drink' 대신 다른 단어를 써도 자연스러워?"`

### 2-2. 데이터 구조 설계 (`msg.feedback` 확장)
* 각 사용자 메시지의 `msg.feedback` 객체 내부에 추가 질의응답 배열(`qa_threads`)과 로딩 상태(`is_qa_loading`)를 추가함:
  ```javascript
  msg.feedback = {
    is_open: true,
    is_loading: false,
    target_used: false,
    evaluation: "...",
    nuance_tip: "...",
    recommended_korean: "...",
    better_alternative: "...",
    saved_to_db: false,
    is_editing: false,
    is_qa_loading: false,
    qa_threads: [
      {
        qa_id: "qa_1794...",
        question: "술을 안 마신다고 정중하게 말하려면 어떻게 해?",
        answer: "비즈니스나 모임 자리에서 분위기를 깨지 않으면서 정중히 논알코올 음료를 요청할 때는 아래와 같이 표현하시면 매우 자연스럽습니다.",
        suggested_korean: "저는 오늘 알코올은 사양할게요. 탄산수에 라임 띄워서 주시겠어요?",
        suggested_english: "I'm sticking to non-alcoholic drinks tonight—could I just get a sparkling water with lime?",
        saved_to_db: false,
        timestamp: 1794146500000
      }
    ]
  };
  ```

### 2-3. UI/UX 설계 (`renderChatMessages` 내 `feedbackCardHtml`)
1. **피드백 카드 하단 `💬 코치에게 추가 질문하기` 영역 신설:**
   * 추천 대안 문장 박스(`🇰🇷 / 🇺🇸`) 바로 아래에 구분선을 두고 **추가 질문 입력창(`#fbQaInput_{msg_id}`) + `[질문하기]` 버튼(`#btnFbQaSend_{msg_id}`)**을 배치함(`Enter` 키로 즉시 전송 지원).
2. **질의응답 스레드(`qa_threads`) 렌더링:**
   * 사용자가 질문하면 피드백 카드 내부에 **`Q. 내 질문`**과 **`A. 코치의 상세 답변`**이 순차적으로 쌓임(연속 질문 가능).
   * 코치의 답변에 새로운 추천 영어/한글 표현(`suggested_english`, `suggested_korean`)이 포함된 경우, 답변 카드 내부에 **`[🔊 발음 듣기]`** 및 **`[⭐ 이 표현도 오답노트에 저장]`**(`saveFollowUpExpressionToDB`) 버튼을 함께 제공하여 추가 질문으로 얻은 표현도 즉시 플래시카드(오답노트)에 저장할 수 있도록 지원함.

### 2-4. AI 프롬프트 및 호출 함수 설계 (`buildFeedbackFollowUpPrompt`, `callFeedbackFollowUpAI`, `askFeedbackFollowUpQuestion`)
* **문맥(Context) 전달:**
  * 현재 롤플레잉 시나리오(`session.title`, `session.business_context`, `session.partner_persona`)
  * 직전 상대 파트너의 영어 대사(`prevPartnerMsg`) 및 내가 입력했던 영어 문장(`msg.text`)
  * 최초 코칭 피드백 내용(`evaluation`, `nuance_tip`, `better_alternative`) 및 이전 추가 질의응답 내역(`qa_threads`)
  * 사용자가 새로 입력한 질문(`userQuestion`)
* **응답 JSON 스키마:**
  ```json
  {
    "answer": "사용자의 추가 질문에 대한 친절하고 명확한 원어민 비즈니스 코치의 설명 (한국어)",
    "suggested_korean": "추가 질문과 관련해 추천할 만한 새로운 영어 문장이 있을 경우 그에 대응하는 자연스러운 한국어 문장 (없으면 빈 문자열)",
    "suggested_english": "추가 질문과 관련해 추천할 만한 새로운 원어민 영어 문장 (없으면 빈 문자열)"
  }
  ```
* `callGeminiWithAutoModelDiscovery` 고속 모델 엔진을 그대로 사용하여 **1~1.5초 내에 즉시 코치 답변이 달리고 Firestore 클라우드에 자동 저장**되도록 구현함.

---

## 📊 3. 사전 영향도 평가 (Impact Assessment)

| 점검 대상 | 영향도 | 상세 내용 |
| :--- | :---: | :--- |
| **GNB / 로그인 / 모달 시스템** | **없음 (안전)** | 기존 인증 및 모달 레이아웃 변경 없음 |
| **기존 피드백 카드 및 오답노트** | **완전 호환** | 기존 `msg.feedback` 구조를 그대로 유지하며 하단에 `qa_threads` 영역만 확장됨 (`qa_threads`가 없는 기존 메시지도 에러 없이 정상 렌더링) |
| **통합 문장 DB & 플래시카드** | **긍정적 개선** | `(영어문장, 한글문장)` 2열 표준화 및 `#0` 제목 행 제외로 `900_1.xlsx`가 `#1~#899`로 정확히 정렬되며, 피드백 추가 질문에서 얻은 표현도 오답노트 저장 가능 |

---

## 🧪 4. 검증 계획 (Verification Plan)
1. **구문 및 태그 무결성 검사:** Node `vm.Script` JS 문법 검증 및 HTML `<div>` 여닫는 태그 쌍 일치 검증.
2. **Chrome E2E 자동화 검증:**
   * **검증 1:** `900_1.xlsx`의 1행(`Title / 해석`)이 자동 제거되고 `"It's my first time"`이 `#1`번으로 정렬되는지 확인.
   * **검증 2:** 피드백 카드 내 추가 질문 입력창에 질문을 입력·전송했을 때 1~2초 내로 AI 코치 답변(`answer` + 추천 영/한 표현)이 렌더링되고 `[⭐ 이 표현도 오답노트에 저장]`까지 정상 작동하는지 확인.

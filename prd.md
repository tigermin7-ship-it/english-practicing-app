# [최종 마스터 PRD] 비즈니스 영어 실전 롤플레잉 & 플래시카드 올인원 웹앱 (BizRoleplay)

---

## 0. 전수 검토 결과 및 핵심 아키텍처 보완 사항 (Executive Review Summary)

전체 대화 히스토리와 기존 PRD를 정밀 대조한 결과, 실제 개발 시 발생할 수 있는 **5가지 핵심 누락 및 기술적 모순점**을 발견하여 본 최종 문서에 완벽히 반영했습니다.

1. **다중 세션 프리셋 및 간헐적 이어하기 복원 (대화 초기 핵심 요구사항 반영):**
   - *기존 한계:* 단순 '새 세션 시작(초기화)' 버튼만 정의되어 있어, 여러 비즈니스 상황과 파트너를 미리 설정해 두고 간헐적으로 세션을 오가며 대화를 이어가는 기능이 누락됨.
   - *보완:* **다중 세션 매니저(`sessionsDB`)**를 도입하여 향후 예정된 미팅별/파트너별 세션을 여러 개 미리 생성·저장하고, 언제든 세션을 전환하여 이전 맥락 그대로 이어서 대화하거나 특정 세션만 초기화할 수 있도록 설계.
2. **AI 파트너의 선제적 대화 시작(AI-First Proactive Trigger) 명세화:**
   - *기존 한계:* 사용자가 먼저 영어로 말을 걸어야만 AI가 응답하는 수동적 구조로 기술됨.
   - *보완:* 세션 시작 시(또는 `[🎲 AI가 먼저 질문/상황 던지기]` 클릭 시) 설정된 시나리오와 타깃 영어 표현을 바탕으로 **AI 파트너가 먼저 비즈니스 요청이나 질문을 던져** 사용자의 영어 사고를 즉각 유도하도록 설계.
3. **단일 HTML 파일 제약과 클라우드 서버 동기화(E2EE) 간의 기술적 연결 고리 확립:**
   - *기존 한계:* "서버 없는 단일 `index.html` 파일"이라고만 지시하면 안티그래비티가 클라우드 동기화 로직을 더미(Mock) 코드로 처리할 위험이 큼.
   - *보완:* 별도 백엔드 코딩 없이 단일 `index.html`에서 즉시 작동하는 **Serverless REST API(Firebase Realtime DB / Supabase REST / GitHub Gist API 중 선택 연동) + LocalStorage 캐싱 + 암호화 백업 파일 Import/Export**의 3중 하이브리드 동기화 아키텍처를 구체화.
4. **대량 문장 DB의 스마트 샘플링 주입 (Context Window 최적화):**
   - *기존 한계:* 다중 엑셀 업로드로 수백 개의 문장이 쌓일 경우, 모든 문장을 매 프롬프트에 넣으면 AI의 역할 몰입도가 떨어지고 토큰 비용이 급증함.
   - *보완:* 활성화된 필터(특정 파일 또는 오답노트) 내에서 **아직 사용해보지 않았거나 중요도가 높은 문장 5~10개를 동적으로 샘플링**하여 해당 턴의 롤플레잉 프롬프트에 주입하도록 고도화.
5. **피드백 문장 아카이빙 시 직관적 편집 및 한·영 자동 매칭:**
   - *보완:* 피드백 댓글에서 `[⭐ 오답노트에 저장]` 클릭 시, 피드백 엔진이 생성한 **'자연스러운 한국어 의미 + 교정된 원어민 비즈니스 영어 문장'** 쌍이 한 번에 묶여 `category: "대화피드백"` 태그로 단일 DB에 즉시 추가되며, 사용자가 원할 경우 텍스트를 미세 수정할 수 있는 인라인 기능을 포함.
6. **스마트 텍스트 일괄 붙여넣기(Bulk Text Paste) 및 수기 입력 접근성 강화 (추가 보완):**
   - *기존 한계:* 엑셀/CSV 파일 업로드 위주로 기술되어 있어, 메모장·웹페이지·교재에서 공부할 "영어+한국어" 문장 세트를 즉시 복사해 붙여넣거나 즉석에서 몇 문장만 입력해 연습하려는 경우 번거로움 발생.
   - *보완:* 엑셀 업로드와 동등한 메인 입력 수단으로 **스마트 텍스트 멀티라인 파서(탭, `|`, `-`, `/`, 또는 한·영 줄바꿈 교차 패턴 자동 인식)** 및 단건 빠른 추가 UI를 제공.
7. **학습 카운트(`practice_count`) 자동 갱신 및 미학습 문장 우선 샘플링 (추가 보완):**
   - *기존 한계:* 스키마에 `practice_count`가 정의되어 있으나 증가 조건 및 샘플링 가중치 로직이 누락됨.
   - *보완:* 플래시카드 학습 완료 클릭 시 또는 롤플레잉 피드백에서 목표 표현 활용(`target_used: true`) 판정 시 `practice_count`를 자동 증가시키고, 롤플레잉 프롬프트 주입 시 `is_starred: true` 및 `practice_count`가 낮은 문장을 우선 샘플링하도록 알고리즘 명세화.
8. **API 응답 안정성(`responseSchema`) 및 음성 입출력(TTS 속도 조절 / STT 마이크) 보강 (추가 보완):**
   - *보완:* Gemini API 호출 시 `responseMimeType: "application/json"`과 함께 `responseSchema` 및 마크다운 코드블록 Strip 방어 로직을 이중 적용하고, Web Speech API 기반 TTS 배속 조절(`0.8x`~`1.2x`) 및 채팅창 STT(음성으로 영어 답변 말하기) 버튼을 추가하여 실전 회화 몰입도 극대화.

---

## 1. 프로젝트 개요 및 비전 (Project Overview)

- **제품명:** BizRoleplay (비즈니스 영어 실전 체화 및 인출 훈련 플랫폼)
- **개발 형태:** 단일 파일 웹 애플리케이션 (Single-file `index.html` 내 HTML5, Tailwind CSS, Vanilla JS 통합)
- **핵심 가치:**
  - 외국인과의 접점이 없는 일상에서도 **'영어로 생각하고 즉각 문장화하는 환경'**을 강제성이나 숙제 부담 없이 메신저 대화 형태로 일상화.
  - **사전 인출 훈련(플래시카드) → 실전 적용(무중단 롤플레잉) → 선택적 교정(인라인 댓글 피드백) → 오답 축적(단일 DB 아카이빙)**의 무한 순환 학습 루프 완성.

---

## 2. 개발 배경 및 문제 해결 매트릭스 (Context & Solutions)

| 사용자의 실제 페인포인트 (대화 맥락) | BizRoleplay의 기술적·UX적 해결 방안 |
| :--- | :--- |
| **1. 일상 속 영어 사고의 비지속성:** 영어 일기나 뉴스 동시통역 연습은 숙제처럼 느껴지고 혼자서 지속하기 어려움 | **AI 선제 발화(Proactive Prompting) & 다중 세션:** AI 파트너가 비즈니스 상황을 가정해 먼저 질문과 요청을 던져 자연스럽게 영어 답변을 유도 |
| **2. 기존 AI 챗봇/Gems의 역할 이탈:** 롤플레잉 도중 AI가 설명 모드로 빠지거나 강사처럼 굴어 대화 맥락이 끊김 | **이원화된 프롬프트 엔진(Dual-Prompt Engine):** 대화 생성 API와 피드백 API를 물리적으로 분리. 롤플레잉 엔진은 문법 설명·훈수를 100% 금지 |
| **3. 매 턴 피드백으로 인한 대화 단절:** 답변할 때마다 교정이 끼어들면 실제 대화가 아닌 시험을 치르는 기분이 듦 | **온디맨드 인라인 댓글(Thread) 피드백:** 평소에는 실제 메신저처럼 대화만 이어가고, 내가 확인하고 싶은 내 말풍선에만 `[💬 피드백]` 버튼을 눌러 댓글로 확인 |
| **4. 파편화된 엑셀 자료와 교정 문장 휘발:** 여러 엑셀 파일을 관리하기 번거롭고, 채팅 중 배운 좋은 표현이 날아감 | **단일 통합 DB & 원클릭 오답노트:** 다중 엑셀 파일과 실시간 채팅 피드백 문장을 단일 DB로 통합 관리하고 태그(`파일명`, `대화피드백`)별로 필터링 |
| **5. 사전 문장 암기 및 영작 인출 연습 필요:** 대화 전후로 내가 등록한 한·영 문장 세트를 빠르게 암기·테스트하고 싶음 | **전체 화면 2단계 분할 플래시카드 모달:** 상단 50%에 한글 노출 → 생각 후 클릭 시 하단 50%에 영문 노출. `[✕ Exit]` 클릭 시 원래 대화창으로 즉시 복귀 |
| **6. 기밀 정보 노출 우려 vs 기기 간 동기화:** 실제 비즈니스 시나리오 유출은 막아야 하지만, PC와 모바일 간 연동 및 캐시 삭제 방지가 필요함 | **Zero-Knowledge 종단간 암호화(E2EE) 동기화:** 브라우저 내에서 마스터 비밀번호로 AES-GCM 암호화 후 클라우드 DB와 실시간 동기화 (서버 해킹 시에도 해독 불가) |

---

## 3. 시스템 아키텍처 및 기술 규격 (Technical Specifications)

- **Single-File Mandate:** 모든 마크업, 스타일, 비즈니스 로직을 단 하나의 `index.html` 파일로 구현.
- **프론트엔드 스택:**
  - **UI/Styling:** Tailwind CSS CDN (`https://cdn.tailwindcss.com`)
  - **Icons:** Lucide Icons CDN (`https://unpkg.com/lucide@latest`)
  - **Excel/CSV Parser:** SheetJS CDN (`https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js`)
- **LLM API 연동 규격:**
  - 기본 지원: Google Gemini API (`gemini-2.5-flash` 또는 `gemini-2.5-pro` 엔드포인트 선택 가능) 및 OpenAI 호환 API.
  - 피드백 엔진 호출 시 `responseMimeType: "application/json"` 옵션을 사용하여 파싱 오류 없는 정형 JSON 응답 보장.
- **보안 및 동기화 스택 (Zero-Knowledge E2EE):**
  - **암호화:** 브라우저 내장 `window.crypto.subtle` (Web Crypto API) 활용.
  - **키 파생:** 사용자 마스터 Passcode + 무작위 Salt(16 bytes) → `PBKDF2` ($100,000$ iterations, `SHA-256`) → `AES-GCM` $256\text{-bit}$ 대칭키 생성.
  - **클라우드 동기화 엔드포인트:** 별도 백엔드 구축 없이 단일 파일에서 동작하도록 **Firebase Realtime Database REST URL(또는 GitHub Gist / Custom Webhook URL)** 입력란을 제공하여 암호화된 페이로드(`ciphertext`, `iv`, `salt`)를 `PUT`/`GET` 통신으로 PC-모바일 간 즉시 동기화. (수동 암호화 파일 `.bizenc` 내보내기/불러오기 동시 지원)

---

## 4. 데이터 스키마 설계 (Data Models)

애플리케이션의 모든 상태는 하나의 최상위 상태 객체(`AppState`)로 관리되며, 통째로 암호화되어 로컬 및 클라우드에 동기화됩니다.

### 4.1 통합 문장 데이터베이스 (`sentencesDB`)
다중 엑셀/CSV 업로드 문장과 대화 중 저장한 피드백 문장을 단일 배열로 통합 관리합니다.
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
    "is_starred": false,
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

### 4.2 다중 롤플레잉 세션 데이터베이스 (`sessionsDB`)
다양한 비즈니스 상황과 파트너를 미리 여러 세션으로 만들어 두고, 간헐적으로 이어서 대화할 수 있는 구조입니다.
```json
[
  {
    "session_id": "sess_001",
    "title": "A사 바이어 John - Q4 단가 인하 방어",
    "partner_name": "John Miller (Purchasing Director)",
    "partner_persona": "비용 절감 압박을 받고 있어 단가 10% 인하를 강하게 요구하는 노련한 해외 바이어",
    "business_context": "연간 재계약 협상 미팅. 당사는 원자재 가격 상승으로 단가 인하가 불가능하며, 대신 물량 확대 시 마일리지 혜택을 제안해야 함.",
    "difficulty": "Advanced",
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

### 5.1 다중 세션 프리셋 관리 및 시나리오 설정 (Multi-Session & Scenario Manager)
- **세션 목록 및 간헐적 이어하기:**
  - 좌측 사이드바(모바일은 드로어 메뉴)에서 여러 개의 비즈니스 세션을 생성, 선택, 이름 변경, 삭제 가능.
  - 특정 세션을 클릭하면 해당 파트너와의 이전 대화 내역(`messages[]`)과 설정값(페르소나, 비즈니스 상황, 난이도, 연동된 표현 필터)이 즉시 복원되어 언제든 대화를 이어갈 수 있음.
- **세션 생성 및 상세 설정 모달:**
  - **세션 제목:** 예) `투자사 VC 파트너 - 런웨이 및 BEP 질문 방어`
  - **상대 파트너 역할(Persona):** 직함, 성향, 말투, 이해관계 입력.
  - **비즈니스 상황 정보(Context):** 현재 직면한 이슈, 나의 목표, 방어해야 할 논리 등을 상세히 입력.
  - **연습할 표현 범위(Target Filter):** `전체 문장`, 특정 업로드 파일명, 또는 `⭐ 대화피드백(오답노트)` 중 선택.
  - **난이도 조절(Difficulty Selector):**
    - `Beginner (초급)`: 쉬운 어휘, 짧고 명확한 질문, 친절하고 협조적인 파트너 태도.
    - `Intermediate (중급)`: 일반 실무 미팅 속도, 자연스러운 비즈니스 관용구 사용.
    - `Advanced (고급)`: 압박 질문, 날카로운 반론, 까다로운 협상 태도 및 고급 비즈니스 어휘 구사.
- **세션 초기화 vs 새 세션 생성 분리:**
  - `[🔄 현재 대화 비우기 (Reset History)]`: 파트너 및 상황 설정은 그대로 유지한 채, 현재 세션의 대화 기록만 깨끗이 비우고 처음부터 다시 연습.
  - `[➕ 새 시나리오 세션 만들기]`: 완전히 새로운 비즈니스 상황 세션을 추가 생성.

### 5.2 무중단 메신저형 롤플레잉 대화창 (Uninterrupted Roleplay Interface)
- **AI 선제 발화 기능 (AI-First Trigger):**
  - 대화 내역이 비어 있는 세션에 진입하거나 `[🎲 AI 파트너가 먼저 말 걸기]` 버튼을 누르면, AI가 현재 시나리오와 타깃 표현을 바탕으로 **실제 외국인 파트너처럼 먼저 첫인사와 함께 핵심 비즈니스 질문/요청을 던짐.**
  - 시나리오를 미처 길게 적지 않은 상태에서도 타깃 영어 문장만 있으면 AI가 그 문장을 쓸 수밖에 없는 비즈니스 상황을 즉석에서 가정하여 질문을 던짐.
- **역할 엄격 고정 (Zero-Explanation Policy):**
  - 사용자가 영어로 답변을 전송하면, AI는 **절대 문법을 지적하거나 상황을 제3자 시점에서 해설하지 않음.**
  - 오직 설정된 비즈니스 파트너의 입장에서 사용자의 말에 반응하고 다음 질문이나 협상 조건을 제시함.
- **타깃 표현 힌트 서랍 (Optional Hint Drawer):**
  - 대화 도중 내가 써야 할 목표 표현이 잘 떠오르지 않을 때, 입력창 상단의 `[💡 타깃 표현 힌트 보기]` 토글을 열면 이번 세션의 목표 문장(한글 또는 한+영)을 슬쩍 참고할 수 있음.
- **음성 입출력 지원 (STT 마이크 입력 & 파트너 메시지 TTS):**
  - 메시지 입력창 우측에 `[🎙️ 음성 입력]` 버튼을 배치하여 Web Speech API(`SpeechRecognition`, `en-US`)를 통해 실제 입으로 영어를 말해 텍스트로 입력할 수 있도록 지원.
  - 파트너(AI) 말풍선 옆에도 `[🔊 발음 듣기]` 버튼을 제공하여 리스닝 훈련 병행 가능.

### 5.3 온디맨드 인라인 댓글 피드백 & 원클릭 오답노트 저장
- **선택적 피드백 요청:**
  - 사용자가 보낸 모든 영어 말풍선 우측 하단에 작고 깔끔한 `[💬 피드백]` 버튼 배치.
  - 버튼을 누르지 않으면 아무런 교정 없이 대화가 계속 이어짐.
- **댓글(Thread) 형태 인라인 출력:**
  - `[💬 피드백]` 클릭 시, 대화 흐름을 끊거나 화면을 이동하지 않고 **해당 말풍선 바로 아래에 접이식(Accordion) 댓글 카드**가 열리며 분석 결과 표시:
    1. **목표 표현 활용 여부 배지:** 타깃 표현 적용 성공 여부(`target_used`) 표시 및 사용된 타깃 문장의 `practice_count` 자동 +1 갱신.
    2. **한 줄 코칭 (Nuance & Grammar):** 문법 오류나 비즈니스 격식(Tone & Manner)에 대한 1~2문장 핵심 코멘트.
    3. **추천 원어민 표현 (Better Alternative):** 해당 맥락에서 가장 자연스럽고 세련된 비즈니스 영어 문장 및 매칭되는 한글 의미 제시.
- **원클릭 통합 DB 저장 (`[⭐ 오답노트에 추가]`):**
  - 피드백 댓글 내 추천 문장 우측에 `[⭐ 오답노트에 추가]` 버튼 제공 (저장 전 한/영 텍스트를 사용자가 직접 수정할 수 있는 간단 편집 모드 지원).
  - 클릭 즉시 통합 문장 DB(`sentencesDB`)에 `source_file: "chat_feedback"`, `category: "대화피드백"` 태그를 달아 영구 추가.
  - 추가된 문장은 즉시 플래시카드 모드의 `⭐ 대화피드백(오답노트)` 필터에 반영되어 복습 가능.

### 5.4 스마트 문장 세트 입력 (텍스트 일괄 붙여넣기 + 다중 엑셀/CSV 업로드) 및 단일 DB 관리
- **스마트 텍스트 일괄 붙여넣기 (Bulk Text Paste) 및 단건 빠른 추가:**
  - 파일 없이도 공부할 "영어문장+한국어" 세트를 텍스트창에 바로 복사·붙여넣기하여 등록할 수 있는 `[📝 텍스트로 문장 세트 빠른 추가]` 모달/드로어 제공.
  - `탭(\t)`, `|`, `-`, `:` 구분자 방식(예: `If you increase... | 주문 물량을...`) 및 줄바꿈 교차 방식(홀수 줄 영어, 짝수 줄 한국어 또는 한글/영어 문자 비율 자동 판별)을 모두 지능적으로 파싱하여 세트 이름(`source_file` 태그)과 함께 일괄 등록.
- **다중 파일 일괄 업로드 및 자동 컬럼 매핑:**
  - 복수의 `.xlsx`, `.xls`, `.csv` 파일을 드래그 앤 드롭으로 한 번에 업로드 가능.
  - SheetJS를 통해 파싱하며, 열 제목이 `한글/영어`, `Korean/English`, `KR/EN`, `의미/표현` 등 다양하거나 헤더가 없더라도(한글 포함 비율 검사를 통해 한국어/영어 열 자동 판별) 지능적으로 매핑.
- **단일 DB 병합 및 출처 태깅:**
  - 업로드된 모든 문장은 단일 `sentencesDB` 배열로 병합(중복 영어 문장은 자동 중복 제거 또는 업데이트)되며, 파일명 또는 입력한 세트명이 `source_file` 태그로 자동 부여됨.
- **데이터 관리 및 엑셀 내보내기 (Export):**
  - 등록된 문장 목록을 태그별로 조회·검색·인라인 수정·개별 삭제하거나 직접 문장 쌍을 수기 입력할 수 있는 관리 뷰 제공.
  - 대화 중 누적된 `⭐ 대화피드백` 문장을 포함한 전체 또는 특정 카테고리 DB를 언제든 `.xlsx` 엑셀 파일로 다시 다운로드 가능.

### 5.5 전체 화면 2단계 분할 플래시카드 모드 (Full-Screen Flashcard Overlay)
- **독립 모달 진입 및 완벽한 복귀:**
  - 상단 헤더의 `[🎴 플래시카드 암기]` 버튼 클릭 시, 기존 대화창 위에 전체 화면(`100vw`, `100vh`, `z-index: 50`) 모달 오버레이로 실행.
  - 우측 상단 `[✕ Exit (대화로 돌아가기)]` 버튼(또는 `ESC` 키)을 누르면 진행 중이던 롤플레잉 대화 스크롤 위치와 입력 중이던 텍스트까지 100% 보존된 상태로 즉시 복귀.
- **학습 대상 필터링:**
  - 상단 바에서 `전체 문장`, `[특정 세트/엑셀 파일명]`, `⭐ 대화피드백(오답노트)`, `★ 중요 표시(Starred)`, `🔥 미암기 집중(practice_count == 0)` 중 원하는 범위만 골라 학습 가능.
- **상·하단 50:50 분할 2단계 인출(Retrieval) 레이아웃:**
  - **화면 상단 절반 (Top 50%):**
    - 선택된 카드의 **한글 문장**만 큼직하고 가독성 높은 폰트로 정중앙에 노출 (출제 방향 토글 `[한➔영 / 영➔한]` 옵션 지원, 기본값은 `한➔영`).
    - 사용자는 이 한글 문장을 보고 영어로 어떻게 말할지 머릿속으로 영작하거나 입 밖으로 말해봄.
  - **화면 하단 절반 (Bottom 50%):**
    - **1단계 (대기 상태):** 영어 문장이 가려진 상태로 *"화면을 클릭(탭)하거나 Space바를 눌러 영어 문장 확인"* 안내 영역 표시.
    - **2단계 (노출 상태):** 화면 어디든 한 번 더 클릭(탭)하면 하단 절반 영역에 **정답 영어 문장**과 관련 메모(`notes`)가 선명하게 노출됨 (원어민 발음 듣기 `🔊 TTS` 및 배속 `0.8x / 1.0x / 1.2x` 선택, `[✅ 암기 완료 (+1)]` 카운트 버튼 제공).
- **카드 내비게이션 및 학습 편의 기능:**
  - 하단 컨트롤 바 및 단축키 지원: `[◀ 이전 카드 (←)]`, `[정답 확인 / 다음 단계 (Space / 화면 클릭)]`, `[다음 카드 (→) ▶]`, `[🔊 발음 듣기 (S)]`, `[★ 별표 토글 (B)]`.
  - `[🔀 셔플(랜덤 순서)]` 토글 및 `[★ 암기 미숙/중요 별표]` 체크 기능 제공.

### 5.6 Zero-Knowledge 종단간 암호화(E2EE) 및 기기 간 동기화
- **마스터 비밀번호(Passcode) 기반 잠금 및 빠른 시작(Quick-Start) 지원:**
  - 최초 실행 시 바로 앱을 체험할 수 있도록 로컬 스토리지 기반 즉시 시작을 지원하되, 상단 `[☁️ 보안/동기화]`에서 마스터 Passcode를 설정하면 `AES-GCM 256-bit` 암호화 잠금 모드가 활성화됨.
  - 모든 시나리오, 파트너 정보, 대화 내역, 문장 DB 및 API Key는 마스터 Passcode 설정 시 평문(Plaintext)으로 외부 서버에 절대 전송되지 않으며, 오직 브라우저 메모리 내에서 `AES-GCM 256-bit`로 암호화된 암호문(Ciphertext) 형태로만 저장됨.
- **PC ↔ 모바일 심리스 동기화 및 캐시 삭제 대비:**
  - 사용자가 설정 패널에 본인의 경량 클라우드 DB URL(예: 무료 Firebase Realtime DB URL 또는 GitHub Gist Token)을 1회 입력해 두면, `[☁️ 클라우드 업로드(Push)]` / `[📥 클라우드 불러오기(Pull)]` 클릭 시 암호화된 데이터 블록이 클라우드와 동기화됨.
  - 모바일 스마트폰에서 접속하거나 브라우저 캐시/인터넷 기록을 모두 삭제하더라도, 동일한 동기화 URL과 마스터 Passcode만 입력하면 모든 세션과 오답노트가 1초 만에 완벽히 복원됨.
  - 클라우드 URL 설정 없이도 쓸 수 있도록 `[💾 암호화 백업 파일 저장(.bizenc.json)]` 및 `[📂 백업 파일 불러오기]` 버튼도 기본 내장.

---

## 6. 화면 레이아웃 및 UI/UX 구조 (Wireframe Blueprint)

### 6.1 메인 롤플레잉 워크스페이스 (PC/모바일 반응형)
```text
+-------------------------------------------------------------------------------------------------------+
| [☰] BizRoleplay | 현재 세션: 바이어 John 단가협상 [Mid] | 타깃: [단가협상.xlsx ▼] | [🎴플래시카드] [☁️보안동기화] |
+-------------------------------+-----------------------------------------------------------------------+
| [좌측 사이드바: 세션 & DB]     | [상단 시나리오 요약 바 (접기/펼치기)]                                  |
|                               | 👤 파트너: John (해외 바이어) | 📋 상황: Q4 단가 10% 인하 방어 및 물량 유도 |
| [➕ 새 롤플레잉 세션 추가]     | [🎲 AI가 먼저 질문 던지기]  [💡 타깃 표현 힌트(15)]  [🔄 현재 대화 비우기] |
|                               +-----------------------------------------------------------------------+
| 📂 내 롤플레잉 세션 목록       | [메신저형 롤플레잉 대화 영역]                                         |
|  • [활성] 바이어 John 단가협상 |                                                                       |
|  • VC 투자유치 피칭 방어       | 👤 Partner (John):                                                    |
|  • 본사 임원 납기지연 해명     | "We need a 10% price cut for Q4. How much can you offer?"             |
|                               |                                                                       |
| ----------------------------- |                                      🙋‍♂️ Me:                           |
| 📊 통합 문장 DB 관리 (총 84문장)|         "10% is hard. If you order more, we give 5% discount."        |
|  [📁 다중 엑셀/CSV 업로드 +]   |                                               [💬 피드백 확인 (열림▲)] |
|  • 전체 문장 (84)              |         +-----------------------------------------------------------+ |
|  • ⭐ 대화피드백 오답노트 (12) |         | ✨ [원어민 코칭 피드백]                     [🎯 타깃표현 사용] | |
|  • 단가협상.xlsx (32)          |         | • 평가: 의미는 통하나 'hard', 'give'보다 격식 표현 권장    | |
|  • 이메일필수.csv (40)         |         | • 한글: 주문 물량을 늘려주신다면 5% 할인을 검토하겠습니다.  | |
|  [📥 전체 DB 엑셀 내보내기]    |         | • 추천: "If you increase the order volume, we can consider | |
|                               |         |          an additional 5% discount."                      | |
| ⚙️ [API 키 및 E2EE 설정]       |         |                              [✏️ 수정] [⭐ 오답노트에 저장] | |
|                               |         +-----------------------------------------------------------+ |
|                               +-----------------------------------------------------------------------+
|                               | [영문 메시지 입력창 (Enter 전송, Shift+Enter 줄바꿈)       ] [전송 ➤] |
+-------------------------------+-----------------------------------------------------------------------+
```

### 6.2 전체 화면 2단계 플래시카드 모드 (`z-index: 50` Modal Overlay)
```text
+-------------------------------------------------------------------------------------------------------+
| 🎴 Flashcard Mode | 범위: [⭐ 대화피드백 오답노트 (12) ▼] | 진행: 04 / 12 | [🔀 셔플 ON] | [✕ Exit 복귀] |
+-------------------------------------------------------------------------------------------------------+
|                                                                                                       |
|                                       [ 상단 50% 영역 : 한글 문장 ]                                     |
|                                                                                                       |
|                        "주문 물량을 늘려주신다면 5% 추가 할인을 검토해 보겠습니다."                      |
|                                                                                                       |
|                                   (머릿속으로 영작해 본 후 아래를 클릭하세요)                           |
+=======================================================================================================+
|                                                                                                       |
|                                   [ 하단 50% 영역 : 영어 정답 (클릭 시 노출) ]                          |
|                                                                                                       |
|            "If you increase the order volume, we can consider an additional 5% discount."             |
|                                                                                                       |
|                     [🔊 발음 듣기 (TTS)]                       [★ 중요 문장 북마크]                    |
|                     💡 메모: [원래 내 문장] If you order more, we give 5% discount.                    |
+-------------------------------------------------------------------------------------------------------+
|              [ ◀ 이전 카드 (←) ]            [ Space / 클릭: 정답 토글 ]            [ 다음 카드 (→) ▶ ]  |
+-------------------------------------------------------------------------------------------------------+
```

---

## 7. 이원화된 AI 프롬프트 엔지니어링 명세 (Dual-Prompt Engine Specs)

안티그래비티가 코드 내에 하드코딩해야 할 2개의 독립 시스템 프롬프트 규격입니다.

### 7.1 엔진 1: 롤플레잉 파트너 전용 프롬프트 (Roleplay Engine)
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

### 7.2 엔진 2: 온디맨드 댓글 피드백 전용 프롬프트 (Feedback Engine)
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
  "target_used": boolean (true if the user successfully used or closely adapted any registered target expression),
  "matched_sentence_id": "string or null (ID of the matched target expression from the list if target_used is true)",
  "evaluation": "한국어로 작성된 1~2문장의 핵심 피드백 (비즈니스 격식, 문법 정확성, 뉘앙스 적절성 평가)",
  "recommended_korean": "추천 영어 대안 문장에 정확히 대응하는 자연스러운 비즈니스 한국어 문장 (오답노트 플래시카드 상단 노출용)",
  "better_alternative": "Polished, natural, native-level business English sentence that best fits the user's intent in this context (오답노트 플래시카드 하단 노출용)",
  "nuance_tip": "왜 이 대안 표현이 비즈니스 상황에서 더 효과적인지 설명하는 짧은 팁 (한국어)"
}
```

---

## 8. 에지 케이스(Edge Cases) 및 예외 처리 규격

1. **엑셀/CSV 파일 인코딩 및 포맷 예외 처리:**
   - 한글 CSV 파일 업로드 시 흔히 발생하는 `EUC-KR` / `UTF-8` 깨짐 현상을 방지하기 위해 `SheetJS` 옵션에 `codepage: 65001` 및 `TextDecoder('euc-kr')` 폴백(Fallback) 로직을 적용할 것.
   - 빈 행(Empty rows)이나 한쪽 언어만 입력된 불완전한 행은 자동으로 필터링하고 업로드 완료 토스트에 `"총 N개 문장 중 유효한 M개 문장이 병합되었습니다"`라고 안내할 것.
2. **피드백 중복 호출 방지 및 캐싱:**
   - 이미 피드백을 받아 `msg.feedback` 객체가 생성된 말풍선은 `[💬 피드백]` 재클릭 시 API를 다시 호출하지 않고 기존 댓글 스레드를 즉시 열기/닫기(Toggle)할 것 (원할 경우 `[🔄 재분석]` 소형 버튼 제공).
3. **초기 실행 시 샘플 데이터(Onboarding Preset) 기본 제공:**
   - 앱을 처음 열었을 때 빈 화면이 나오지 않도록, 비즈니스 핵심 표현 10문장(단가 협상, 일정 조율, 이메일 요청 등)과 기본 롤플레잉 세션 2개가 초기 상태로 내장되어 즉시 테스트해 볼 수 있도록 구현할 것.
4. **API Key 미설정 시 오프라인 시뮬레이션(체험 모드) 폴백 지원:**
   - 사용자가 아직 Gemini/OpenAI API Key를 입력하지 않은 상태에서도 앱의 롤플레잉 흐름과 인라인 피드백 UI를 즉시 체험할 수 있도록, 타깃 문장 기반 스마트 시뮬레이션 응답 폴백 및 API 설정 안내 배너를 제공할 것.

---

## 9. 안티그래비티(Anti-Gravity) 원클릭 개발 지시문

안티그래비티에게 본 PRD 파일을 업로드한 뒤 아래 프롬프트를 그대로 복사해 전달하세요.

> **[안티그래비티 입력용 프롬프트]**
> 첨부한 `prd_final.md` 문서의 모든 아키텍처, 데이터 스키마, UI/UX 요구사항을 100% 반영하여, 단 하나의 파일(`index.html`)로 완벽하게 동작하는 비즈니스 영어 롤플레잉 & 플래시카드 웹 애플리케이션을 구현해 줘.
>
> **구현 시 반드시 지켜야 할 핵심 체크리스트:**
> 1. **Single-File 완결성:** 외부 빌드 도구 없이 `index.html` 파일 하나에 HTML, Tailwind CSS(CDN), Lucide Icons(CDN), SheetJS(CDN), Vanilla JS를 모두 통합하고, 생략(`// TODO` 등) 없이 전체 동작 코드를 끝까지 작성할 것.
> 2. **다중 세션 프리셋 및 이어하기:** 여러 개의 비즈니스 상황/파트너 세션을 생성·저장하고 세션을 오가며 대화를 이어갈 수 있는 멀티 세션 매니저와, AI 파트너가 먼저 대화를 걸어오는 `[🎲 AI가 먼저 질문 던지기]` 기능을 구현할 것.
> 3. **Dual-Prompt 역할 격리:** 일반 롤플레잉 응답 시에는 절대 문법 설명이나 훈수를 두지 않고 100% 비즈니스 파트너로서만 대답하며, 내 말풍선 아래의 `[💬 피드백]` 버튼을 눌렀을 때만 댓글(Thread) 카드로 교정 피드백과 추천 문장을 보여줄 것.
> 4. **단일 통합 DB & 원클릭 오답노트:** 다중 엑셀/CSV 업로드 문장들과 피드백 댓글에서 `[⭐ 오답노트에 저장]`을 누른 문장(`category: "대화피드백"`)이 하나의 통합 DB 배열에서 태그별로 관리·필터링·엑셀 내보내기 되도록 구현할 것.
> 5. **전체 화면 2단계 플래시카드 모달:** 상단 50%(한글 문장) 노출 후 화면 클릭/Space바 입력 시 하단 50%(영어 정답 및 TTS 발음)가 노출되는 전체 화면 오버레이를 구현하고, `[✕ Exit]` 클릭 시 원래 대화창 상태를 그대로 유지하며 복귀할 것.
> 6. **Web Crypto E2EE 보안 동기화:** 마스터 Passcode 기반 `AES-GCM 256-bit` 암호화를 적용하여 로컬 저장소 캐싱, 클라우드 REST URL(Firebase/Gist 등) 동기화, 암호화 백업 파일 Import/Export가 모두 작동하도록 구현할 것.
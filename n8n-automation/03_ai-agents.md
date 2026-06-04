# 03. AI 노드와 에이전트

n8n은 LangChain을 노드로 감싸, LLM·메모리·도구·벡터DB를 캔버스에서 드래그로 잇게 해준다. AI 노드만 70개가 넘는다. 핵심 구조만 잡자.

## 클러스터 노드: 루트 + 서브

AI 쪽은 일반 노드와 연결 방식이 다르다. **루트 노드**(AI Agent, LLM Chain 등) 아래에 **서브 노드**를 매달아 능력을 붙인다. 선이 옆이 아니라 *아래로* 꽂힌다.

```
        [ AI Agent ]   ← 루트 (오케스트레이터)
         /    |    \
  Chat Model  Memory  Tool(s)   ← 서브 노드
```

- **Chat Model** (필수): 실제 LLM. OpenAI, Anthropic(Claude), Google, Ollama(로컬) 등. 여기에 모델명·API 키를 건다.
- **Memory** (선택): 대화 맥락 유지.
- **Tool** (선택, 여러 개): 에이전트가 호출할 수 있는 외부 능력.
- **Output Parser** (선택): 출력을 정해진 JSON 구조로 강제.

## 어떤 AI 노드를 쓸까

| 상황 | 노드 |
| --- | --- |
| 한 번의 LLM 호출 (요약·분류·번역) | **Basic LLM Chain** |
| 출력을 정해진 JSON으로 받고 싶다 | LLM Chain + **Structured Output Parser** |
| 모델이 *스스로 도구를 골라* 여러 단계 수행 | **AI Agent** (= Tools Agent) |
| 문서 검색 기반 답변 | Agent + **Vector Store** + Embeddings (RAG) |

흔한 실수: 단순 요약에 AI Agent를 쓰는 것. 도구 호출이 필요 없으면 **LLM Chain**이 더 단순하고 빠르다.

## AI Agent: 도구 쓰는 LLM

Agent는 "사용자 질문 → 어떤 도구가 필요한지 LLM이 판단 → 도구 호출 → 결과 보고 다시 판단 → 답"의 루프를 돈다. 현재 기본형은 **Tools Agent**로, LLM의 function/tool calling을 그대로 쓴다.

도구로 붙일 수 있는 것: HTTP Request(아무 API나), 검색(SerpAPI/Brave), SQL·Google Sheets, Slack, 그리고 **다른 워크플로우**(Sub-workflow를 도구로). 각 도구의 *설명*을 잘 써줘야 LLM이 언제 부를지 안다.

## Memory: 대화 맥락

| 메모리 | 방식 |
| --- | --- |
| Window Buffer | 최근 N개 메시지만 유지 |
| Summary | 굴러가는 요약으로 압축 |
| Redis / Postgres | 외부 저장 — 세션 키로 구분, 영속 |

기본 메모리는 **세션/실행 간 영속되지 않는다.** 대화를 계속 잇거나 사용자별로 구분하려면 외부 메모리 + 세션 키가 필요하다.

## 시작점 트리거

- **Chat Trigger**: n8n이 띄운 채팅창으로 대화하며 에이전트 테스트.
- **Webhook**: 외부 앱(카톡/Slack/내 서비스)이 보낸 메시지를 입력으로.

## 최소 구성 예 (요약 분류기)

```
Chat Trigger → AI Agent
                 ├─ Chat Model: Claude
                 ├─ Memory: Window Buffer
                 └─ Tool: HTTP Request (사내 검색 API)
```

질문이 들어오면 Agent가 필요 시 검색 도구를 부르고, 메모리로 직전 맥락을 유지하며 답한다.

## 비용·운영 감각

- LLM 호출은 토큰 과금 — 루프 도는 Agent는 호출이 누적된다. 단순 작업엔 Chain으로 호출 수를 줄인다.
- API 키는 노드가 아니라 Credentials에 저장(암호화). 키 노출 주의.

---

다음: [자주 쓰는 자동화 패턴](04_patterns.md).

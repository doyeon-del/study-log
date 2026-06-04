# 04. 자주 쓰는 자동화 패턴

개념을 다 읽었으면, 실제 워크플로우는 결국 이 몇 가지 골격의 변주다. 난이도 순으로.

## 1) 요약 봇 — 트리거 + AI + 출력 한 바퀴

가장 기본. "소스에서 읽기 → AI로 가공 → 어딘가에 쓰기".

```
Schedule (매일 8시) → RSS Read → Basic LLM Chain (3줄 요약) → Slack/Notion 저장
```

- 배우는 것: 트리거, item 반복(기사 여러 개면 LLM이 item마다 돈다), 표현식으로 본문 넘기기.
- 변주: RSS 대신 Gmail/웹스크랩, 출력은 시트/메일.

## 2) 분류 라우터 — AI 판단 + 분기

들어온 요청을 AI가 분류하고, 종류에 따라 다른 처리로 보낸다.

```
Webhook (문의 수신) → LLM Chain + Structured Output Parser (카테고리 JSON)
   → Switch (카테고리별)
       ├─ 환불 → 담당 Slack 채널
       ├─ 버그 → 이슈 생성
       └─ 기타 → 자동응답 메일
```

- 배우는 것: Output Parser로 **구조화된 JSON** 받기, Switch 분기.
- 핵심: 분류 결과를 자유 텍스트가 아니라 정해진 enum으로 받아야 분기가 안정적.

## 3) 스케줄 리포트 — 수집·집계·코멘트

매일/매주 데이터를 모아 AI가 한마디 붙여 보고.

```
Schedule → DB/Sheets 조회 → Aggregate(집계) → LLM Chain(추세 코멘트) → 메일/Notion
```

- 배우는 것: 여러 item을 하나로 묶기(Aggregate), 집계값을 프롬프트에 넣기.

## 4) RAG — 문서 기반 질의응답

내 문서를 벡터로 저장해두고, 질문이 오면 관련 조각을 찾아 근거로 답한다.

```
[적재] 문서 → Text Splitter → Embeddings → Vector Store(저장)
[질의] Chat Trigger → AI Agent
          ├─ Chat Model
          └─ Tool: Vector Store(검색)
```

- 배우는 것: 임베딩·벡터스토어, 검색 결과를 근거로 환각 줄이기.

## 5) 도구 쓰는 에이전트 — 자율 처리

질문을 받고 에이전트가 알아서 도구(API/검색/DB)를 골라 답을 만든다. → [03번 노트](03_ai-agents.md)의 AI Agent.

## 패턴을 고르는 기준

| 필요 | 패턴 |
| --- | --- |
| 정해진 한 단계 가공 | 1) 요약 봇 (LLM Chain) |
| 결과로 흐름을 나눠야 | 2) 분류 라우터 (Output Parser + Switch) |
| 주기적 집계 보고 | 3) 스케줄 리포트 |
| 내 문서 근거로 답 | 4) RAG |
| 다단계·도구 필요 | 5) AI Agent |

대부분은 1~3으로 충분하다. 도구 호출이 정말 필요할 때만 Agent로 올라간다.

---

다음: [설치하고 첫 워크플로우 만들기](05_setup.md).

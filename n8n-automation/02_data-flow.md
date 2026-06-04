# 02. 데이터 흐름 (n8n의 80%)

노드를 잇는다는 건 "앞 노드의 출력 JSON을 뒷 노드가 받아 쓴다"는 뜻이다. 이 데이터 모양을 손에 익히는 게 n8n 학습의 핵심이다.

## items: 모든 데이터는 배열

노드 사이를 흐르는 건 **items 배열**이다. 각 item은 이렇게 생겼다:

```json
[
  { "json": { "name": "도연", "score": 90 }, "binary": {} },
  { "json": { "name": "민수", "score": 75 } }
]
```

- 실제 데이터는 `json` 안에 들어있다. 파일·이미지는 `binary`.
- **노드는 item마다 한 번씩 실행된다.** 위처럼 2개가 들어오면, 뒤 노드(예: 메일 보내기)는 2번 돈다. 이게 n8n의 반복 방식 — 따로 for문을 안 짜도 item 수만큼 돈다.

## 표현식: 앞 노드 값 끌어오기

노드 입력칸에 `{{ }}`를 쓰면 그 안은 자바스크립트 식으로 평가된다. **지금 처리 중인 item**을 가리키는 게 `$json`:

```
{{ $json.name }}            // 현재 item의 name
{{ $json["score"] }}        // 같은 값, 대괄호 표기
{{ $json.score * 1.1 }}     // 계산도 됨
{{ $now }}                  // 현재 시각
```

특정 노드의 출력을 콕 집어 참조:

```
{{ $('노드이름').item.json.score }}    // 그 노드의 (대응) item
{{ $('노드이름').first().json.score }} // 그 노드의 첫 item
{{ $('노드이름').all() }}              // 그 노드의 전체 items 배열
```

> `$node["이름"]` 같은 옛 표기도 보이지만, 최근 권장은 `$('노드이름')` 함수형이다.

## 자주 쓰는 변형 노드

| 노드 | 하는 일 |
| --- | --- |
| Edit Fields (Set) | 필드 추가/이름변경/정리 — 다음 노드가 원하는 모양으로 다듬기 |
| Filter | 조건 안 맞는 item 버리기 |
| IF / Switch | 조건으로 흐름 분기 |
| Merge | 두 갈래 데이터 합치기 |
| Aggregate | 여러 item을 하나로 묶기 (배열로) |
| Split Out | 한 item 안의 배열을 여러 item으로 펼치기 |
| Code | 위로 안 되면 JS/Python으로 직접 가공 |

## Code 노드 두 모드

- **Run Once for All Items**: `items` 전체를 받아 한 번 실행. 집계·정렬에.
- **Run Once for Each Item**: item마다 실행, `$json`으로 현재 것 접근.

반환은 항상 items 형태로:

```javascript
// 전체 모드 예시
return items.map(i => ({ json: { name: i.json.name, pass: i.json.score >= 80 } }));
```

## 막히면

- 노드를 실행하고 출력 탭에서 **실제 JSON 모양**을 본다. 표현식은 그 키 이름을 그대로 쓴다.
- "왜 안 끌려오지?" → 십중팔구 키 이름 오타이거나, item이 비어 있거나, 참조한 노드가 아직 실행 안 됨.

---

다음: [AI 노드와 에이전트](03_ai-agents.md).

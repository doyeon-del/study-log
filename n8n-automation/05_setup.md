# 05. 설치 — 실습 들어갈 때

개념을 잡았으면 손으로 돌려본다. 가볍게 맛보려면 npx, 계속 쓸 거면 Docker.

## A. 바로 체험 (npx)

Node.js 18+ 필요.

```bash
npx n8n
```

받아서 띄워지면 브라우저로 `http://localhost:5678` 접속. 워크플로우는 로컬에 저장된다. 끄면 끝 — 가볍게 둘러볼 때.

## B. Docker 한 컨테이너 (가장 흔한 시작)

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n
```

- `-v n8n_data:...` 로 볼륨을 걸어야 컨테이너를 지워도 워크플로우·자격증명이 남는다. (이게 없으면 매번 초기화)
- 역시 `http://localhost:5678`.

## C. Docker Compose (오래 쓸 셋업)

DB를 Postgres로 빼서 안정적으로 운영. `docker-compose.yml`에 n8n + Postgres(+ 필요시 Redis)를 한 묶음으로 선언한다. 공식 [n8n-hosting 레포](https://github.com/n8n-io/n8n-hosting)에 패턴별 compose 파일이 있으니 거기서 가져다 쓴다.

## 꼭 챙길 것

- **`N8N_ENCRYPTION_KEY` 백업.** 자격증명(API 키 등)이 이 키로 암호화된다. 키를 잃으면 저장된 모든 자격증명을 영영 못 푼다. Docker는 첫 실행 때 자동 생성되므로, 볼륨/환경변수로 고정하고 따로 백업.
- **Webhook URL.** 외부에서 호출하는 트리거를 쓰려면 n8n이 접근 가능한 주소여야 한다. 로컬 테스트는 터널(예: n8n 내장 tunnel, cloudflared) 필요.
- **타임존.** 스케줄이 의도한 시간에 돌도록 `GENERIC_TIMEZONE`(예: `Asia/Seoul`) 설정.

## 첫 워크플로우로 추천

[04번 노트](04_patterns.md)의 **1) 요약 봇** — Schedule(또는 Manual) → 소스 읽기 → LLM Chain 요약 → 출력. 트리거·item 반복·표현식·AI 노드를 한 바퀴에 다 만진다.

> LLM 노드를 쓰려면 모델 제공자(OpenAI/Anthropic 등) API 키가 필요하다. 로컬만으로 가볍게 하려면 Ollama Chat Model로 로컬 모델을 붙일 수도 있다.

## 막히면

- 공식 문서: https://docs.n8n.io — 노드별 입출력과 옵션이 정확.
- 노드 실행 후 출력 JSON을 직접 보며 [02번 노트](02_data-flow.md)의 표현식으로 끌어온다.

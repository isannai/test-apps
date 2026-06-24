# isann test-apps — M4~M7 test fixtures

이 트리는 **그대로 GitHub repo 로 복사해서 push** 하기 위한 fixture 입니다. 권장 repo: `isannai/test-apps` (public).

```bash
git clone git@github.com:isannai/test-apps.git
# 이 디렉토리 (docs/todo/apps/test/git/) 의 내용을 test-apps/ 루트로 복사
git add -A
git commit -m "init test fixtures (M4-M7)"
git push
```

push 후 raw URL 형태:
```
https://raw.githubusercontent.com/isannai/test-apps/main/<author>/<appname>/<version>/<file>
```

예:
```
https://raw.githubusercontent.com/isannai/test-apps/main/alice/rag/0.1.0/install.ian
https://raw.githubusercontent.com/isannai/test-apps/main/alice/rag/0.1.0/tools.json
https://raw.githubusercontent.com/isannai/test-apps/main/alice/rag/0.1.0/docker-compose.yaml
```

## 트리 구조

```
.
├── alice/
│   └── rag/
│       ├── 0.1.0/                ← 기본 정상 케이스
│       │   ├── install.ian
│       │   ├── tools.json
│       │   └── docker-compose.yaml
│       └── 0.1.1/                ← 멀티버전 (M2 supersede 검증)
│           ├── install.ian
│           ├── tools.json
│           └── docker-compose.yaml
├── bob/
│   └── rag/
│       └── 0.1.0/                ← 동명 다른 author (M0 namespace 충돌 회귀)
│           ├── install.ian
│           ├── tools.json
│           └── docker-compose.yaml
└── malicious/
    └── badcompose/
        └── 0.1.0/                ← M3 lint 거부 시나리오
            ├── install.ian
            ├── tools.json
            └── docker-compose.yaml
```

## 무엇을 검증하나

| 시나리오 | 사용 fixture | 검증 milestone |
|---|---|---|
| 단일 파일 pull (tools.json) | alice/rag/0.1.0/tools.json | M4 |
| install.ian 부트스트랩 | alice/rag/0.1.0/install.ian | M6 |
| 멱등 재설치 거부 | 같은 alice/rag/0.1.0 두 번 | M6 |
| `--force` 덮어쓰기 | alice/rag/0.1.0 + `--force` | M6 |
| 멀티버전 enable | alice/rag/0.1.0 + 0.1.1 | M2/M6 |
| supersede 알림 | alice/rag/0.1.1 enable → 0.1.0 disable | M2 |
| 동명 다른 author 공존 | alice/rag + bob/rag | M0/M6 |
| wire 이름 충돌 없음 | `alice__rag__search` vs `bob__rag__search` | M0 |
| compose lint 거부 | malicious/badcompose | M3/M5 |
| operator override | `policy add --rule compose malicious_badcompose@0.1.0` | M3 |

## 보안 주의

`malicious/badcompose/` 는 의도적으로 **lint 가 거부할 compose** 를 담고 있습니다 (`privileged: true`, host bind, host network). **실제 노드에 install 하지 마세요** — `--force` 와 `policy add --rule compose ...` 를 동시에 박으면 통과해버립니다. M3 회귀 테스트 용도로만 쓸 것.

repo 를 public 으로 둘 거면 README 에 위 경고를 한 번 더 박는 게 좋습니다.

## 이미지 출처

모든 정상 compose 는 **Docker Hub public 이미지** (qdrant/qdrant, redis) 만 참조 — 별도 GHCR push 불필요. 자체 빌드 이미지가 필요해지면 ghcr.io/isannai/<name> 로 push 후 compose 의 image: 만 교체하면 됩니다.

## 다음 (M4-M6 코드 측)

- M4 구현 후: `isann app pull https://raw.githubusercontent.com/isannai/test-apps/main/alice/rag/0.1.0/tools.json --name alice_rag@0.1.0` 가 단일 파일 케이스로 성공해야
- M6 구현 후: hub 부재 동안 ref→URL 해석을 위 raw URL 패턴으로 임시 하드코딩하면 `isann app install alice/rag@0.1.0` 가 end-to-end 동작

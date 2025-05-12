---
layout: post
category: k8s
---

## kubectl top

### 실행 중인 pod, node 의 지표 확인 


Kubernetes에서는 기본적으로 `kubectl top` 명령어를 사용하여 CPU 및 메모리 사용량 지표를 확인할 수 있습니다.

- `kubectl top nodes`: 노드별 리소스 사용량
- `kubectl top pod` : 파드 별 리소스 사용량

해당 명령어 뒤에, 특정 노드 및 파드의 이름을 적게 된다면, 해당 파드 및 노드의 사용량만 확인 해볼 수 있습니다.

```sequence
sequenceDiagram
    participant User
    participant API_Server
    participant Metrics_Server
    participant Kubelet

    User->>API_Server: kubectl top nodes
    API_Server->>Metrics_Server: 요청 전달
    Metrics_Server->>Kubelet: 주기적 수집 (사전 수집)
    Metrics_Server->>API_Server: 노드 메트릭 응답
    API_Server->>User: 메트릭 정보 출력
```
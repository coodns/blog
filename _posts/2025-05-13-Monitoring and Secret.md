---
layout: post
category: k8s
---
## kubectl top

### 실행 중인 pod, node 의 지표 확인 


Kubernetes에서는 기본적으로 `kubectl top` 명령어를 사용하여 CPU 및 메모리 사용량 지표를 확인할 수 있습니다.

- `kubectl top nodes`: **노드별 리소스 사용량**
- `kubectl top pod` : **파드 별 리소스 사용량**

해당 명령어 뒤에, 특정 노드 및 파드의 이름을 적게 된다면, 해당 파드 및 노드의 사용량만 확인 해볼 수 있습니다.

### 지표는 어떻게 가져오는가

<이미지 넣고>
설명도

### ConfigMap, Secret

cfm.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-index
data:
  index.html: |
    hello onboarding
```

sec.yaml
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: nginx-secret
type: Opaque
stringData:
  username: onboarding
  password: welcome
```
*Opaque* 란?
일반적인 용도의 시크릿으로써 ConfigMap과 동일한 목적으로도 사용할 수 있습니다.
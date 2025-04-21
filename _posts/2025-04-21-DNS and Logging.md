---
layout: post
category: k8s
---
### 1. CoreDNS란?

CoreDNS는 쿠버네티스 클러스터의 DNS 역할을 수행할 수 있는, 유연하고 확장 가능한 DNS 서버이며, 클러스터 내에서 주로 내부 도메인 쿼리, 외부 도메인 쿼리에 사용됩니다. 다른 애플리케이션과 마찬가지로 Pod로 호스팅 되며, Deployment 로 실행되어 Service로 요청을 받습니다.&#x20;

CoreDNS는 Kubernetes 클러스터에서 기본적으로 사용되는 DNS 서버입니다. 클러스터 내부에서 서비스 이름을 기반으로 통신할 수 있게 해주는 핵심 컴포넌트로, kube-dns를 대체하여 현재 대부분의 클러스터에서 기본 DNS 애드온으로 사용됩니다



**주요 기능:**

- 클러스터 내 Pod, Service 이름을 IP로 변환 (DNS name resolution)
- DNS 캐싱
- 플러그인 기반 구조로 확장성 뛰어남




### 2. CoreDNS 애드온 설치 및 구성


### 3. 네임스페이스 생성

```bash
kubectl create namespace ts-team
kubectl create namespace sa-team
```

### 4. curl 명령어가 가능한 Pod 배포

ts-team.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: curl-ts
  namespace: ts-team
  labels:
    app: curl-ts
spec:
  containers:
    - name: curl
      image: curlimages/curl
      command: ["sleep", "3600"]
  restartPolicy: Never

```

sa-team.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: curl-sa
  namespace: sa-team
  labels:
    app: curl-sa
spec:
  containers:
    - name: curl
      image: curlimages/curl
      command: ["sleep", "3600"]
  restartPolicy: Never
```

위 파일들을 각각 배포합니다.

```bash
# 네임스페이스 생성
kubectl create namespace ts-team
kubectl create namespace sa-team

# YAML 파일 적용
kubectl apply -f curl-ts.yaml
kubectl apply -f curl-sa.yaml
```

### 5. 각 파드에서 반대편 파드에 ping

```bash
kubectl exec -n ts-team curl-ts -- ping <curl-sa의 IP>
kubectl exec -n sa-team curl-sa -- ping <curl-ts의 IP>
```

### 6. IP 확인 방법

Pod의 IP는 다음 명령어로 확인할 수 있습니다:

```bash
kubectl get pod -o wide -n ts-team
kubectl get pod -o wide -n sa-team
```

각 명령어는 해당 네임스페이스 안의 Pod들의 내부 클러스터 IP를 보여줍니다.

### 7. curl을 통한 IP 없이 통신 시도

```bash
kubectl exec -n ts-team curl-ts -- curl curl-sa.sa-team.svc.cluster.local
kubectl exec -n sa-team curl-sa -- curl curl-ts.ts-team.svc.cluster.local
```

### 8. 어떤 주소를 사용했는가?

Kubernetes DNS는 다음의 이름 구조를 가집니다:

```
<service-name>.<namespace>.svc.cluster.local
```

위 DNS 이름으로 Pod → Service 또는 Pod → Pod 통신이 가능합니다. 위 실습에서는 `curl-sa.sa-team.svc.cluster.local` 과 `curl-ts.ts-team.svc.cluster.local` 을 사용했습니다.

### 9. 통신이 가능했던 이유

CoreDNS는 Kubernetes 내부의 DNS 요청을 처리하여, 위에서 언급한 도메인 형식으로 서비스 이름을 IP로 변환해줍니다. 따라서 실제 IP를 몰라도 해당 DNS 이름을 통해 다른 네임스페이스의 Pod나 Service에 접근할 수 있습니다.




### 10. 같은 네임스페이스 vs 다른 네임스페이스 통신

| 통신 방식     | DNS 이름 예시                                                | 설명                          |
| --------- | -------------------------------------------------------- | --------------------------- |
| 같은 네임스페이스 | `curl-sa`                                                | 같은 네임스페이스일 경우 이름만 사용 가능     |
| 다른 네임스페이스 | `curl-sa.sa-team` 또는 `curl-sa.sa-team.svc.cluster.local` | 다른 네임스페이스는 반드시 전체 도메인 사용 필요 |

### 11. 원리 정리 (Flowchart)

아래는 Pod가 DNS 이름을 사용해 통신할 때의 원리입니다:

![week501](../images/week501.png)

이와 같은 원리로 CoreDNS가 클러스터 내의 통신을 가능하게 해줍니다.

---



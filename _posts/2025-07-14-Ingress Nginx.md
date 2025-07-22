---
layout: post
category: k8s
---
## 1. Ingress Nginx Controller는 무엇인가요?

Ingress Nginx 를 알기 전 Ingress 와 Ingress Controller 부터 알고 가야합니다.

🔹 Ingress란?

Ingress는 Kubernetes에서 클러스터 외부에서 내부 서비스로 들어오는 HTTP 및 HTTPS 트래픽을 라우팅하기 위한 규칙을 정의하는 리소스입니다. 다시 말해, “어떤 도메인으로 들어온 요청을 어떤 서비스로 보낼지”에 대한 정보를 담고 있는 설정 파일이라고 할 수 있습니다.

Ingress는 단순히 정적 규칙만을 정의할 뿐, 실제로 네트워크 요청을 처리하거나 트래픽을 전달하는 역할은 하지 않습니다. 이 역할은 Ingress Controller가 맡습니다.

이러한 Ingress 를 사용하기 위해선 

1. Ingress Controller 설치

	•	Ingress 리소스는 단순한 규칙 정의일 뿐이라, 이를 실행할 Ingress Controller가 반드시 있어야 함

	•	대표적인 선택지: ingress-nginx controller (가장 널리 사용됨),AWS ALB ingress controller

2. Ingress 리소스 정의

	•	트래픽을 어떤 도메인/경로로 받을지, 어떤 서비스로 보낼지를 Ingress 리소스로 정의해야 함.

	•	예: /api 요청은 api-service로, /web은 web-service로 라우팅 등

3. 클러스터 외부에서 접근 가능한 엔드포인트

	•	일반적으로 Ingress Controller는 외부에서 접근 가능해야 하므로, Service 타입이 보통 LoadBalancer 또는 NodePort로 설정됨

	•	AWS라면 LoadBalancer 타입을 쓰면 ALB 컨트롤러 를 통해 NLB 또는 ALB가 자동 생성됨


🔹 Ingress Controller란?

Ingress Controller는 Ingress 리소스에 작성된 규칙을 실제로 동작하게 만들어주는 소프트웨어 컴포넌트입니다.
Kubernetes 클러스터 내에 별도로 배포되며, Ingress 리소스를 실시간으로 감시하고 그에 따라 트래픽 라우팅 동작을 설정합니다.

Ingress Controller는 클러스터 외부로부터의 HTTP(S) 요청을 받아서, Ingress에서 정의한 규칙에 따라 올바른 서비스로 트래픽을 전달합니다.
Ingress 리소스만으로는 아무런 동작도 하지 않으며, 반드시 하나 이상의 Ingress Controller가 클러스터에 존재해야 Ingress가 의미를 가집니다.

Ingress Controller는 여러 종류가 있으며, 각기 다른 방식으로 동작하거나 특정 클라우드 환경에 특화된 기능을 제공하기도 합니다.



🔹 ingress-nginx controller란?

nginx-ingress controller는 가장 널리 사용되는 Ingress Controller 구현체 중 하나로, 내부적으로 NGINX 웹 서버를 기반으로 동작합니다.

Ingress 리소스를 감시하면서 동적으로 NGINX 설정을 생성하고, 이를 통해 들어오는 요청을 적절한 Kubernetes 서비스로 전달합니다.
구성과 배포가 상대적으로 간단하며, 오픈소스 생태계에서 활발히 유지보수되고 있어 실무 환경에서도 자주 사용됩니다.

또한 SSL 인증서 관리, 경로 기반 라우팅, 인증 처리 등 다양한 고급 기능들을 제공하기 때문에 소규모 애플리케이션부터 대규모 서비스까지 폭넓게 활용됩니다.



간단 명료 하게 정리 해보면

외부 트래픽을 전달 받을 경우, ingress 규칙을 보고 ingress controller 가 이의 규칙에 맞는 작업들을 이행해주며,
이와같은 작업들을 해주는 ingress Controller 의 종류 중 하나가 Nginx Ingress Controller 입니다.

## 2. Test 환경 및 시나리오.

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
  namespace: doobiedooba
spec:
  replicas: 1
  selector:
    matchLabels:
      app: api-server
  template:
    metadata:
      labels:
        app: api-server
    spec:
      containers:
      - name: api-server
        image: <myaccount>.dkr.ecr.ap-northeast-2.amazonaws.com/yummy-dummy-backend:api_test
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: doobiedooba
spec:
  selector:
    app: api-server
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
  ```
---


## 4. ALB Ingress Create

먼저 ALB 컨트롤러 를 사용할 경우 다음과 같이 설정 할 경우, ALB 에서는 아래와 같이 리스너 규칙을 통해 / -> /api 로 리다이렉트 작업은 진행 할수 는 있지만, URL 또한 같이 변경됩니다.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apitest-ingress
  namespace: doobiedooba
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]'
    alb.ingress.kubernetes.io/healthcheck-path: /api
    alb.ingress.kubernetes.io/actions.root-redirect: |
      {
        "type": "redirect",
        "redirectConfig": {
          "path": "/api",
          "statusCode": "HTTP_302"
        }
      }
spec:
  ingressClassName: alb
  rules:
  - http:
      paths:
      - path: /
        pathType: Exact
        backend:
          service:
            name: root-redirect
            port:
              name: use-annotation
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend
            port:
              number: 80
```

---


## 3. How to Install It

helm 통해 설치 하였으며, Nginx Inress 컨트롤러 설치시 생성되는 서비스 항목의 대한 타입이 로드밸런서 이기에, 이를 AWS 에서 배포하게 될 경우, ALB 컨트롤러를 통해 생성되도록 구성

```yaml
controller:
  publishService:
    enabled: true
  service:
    type: LoadBalancer
    externalTrafficPolicy: Cluster
    annotations:
      service.beta.kubernetes.io/aws-load-balancer-backend-protocol: http
      service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: '60'
      service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: 'true'
      service.beta.kubernetes.io/aws-load-balancer-ssl-cert: <acm arn>
      service.beta.kubernetes.io/aws-load-balancer-ssl-ports: https
      service.beta.kubernetes.io/aws-load-balancer-type: nlb
    ports:
      http: 80
      https: 443
    targetPorts:
      http: tohttps
      https: http
```



---
## 5. Ingress Nginx Create

```yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apitest-nginx-ingress
  namespace: doobiedooba
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /api
spec:
  ingressClassName: nginx
  rules:
  - host: doobiedooba.chaewoon.me
    http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: backend
              port:
                number: 80
```


/ → Ingres(Controller) → rewrite(Controller) → /api → 서비스 → Pod 응답 "안녕하세요"



위와 같은 방식으로 전달 트래픽이 전달 됩니다. ALB 에서 모든 트래픽 라우팅을 처리하는것과 다르게, 설정해놓은 서비스에 대한 외부 접근 방식에 대해서만, 접근 하도록 만들었습니다.

	1.	클라이언트가 HTTP 요청을 보냄 → 포트 80
	•	targetPorts.http: tohttps에 의해 tohttps 포트에서 301 HTTPS 리디렉트 응답을 보냄
	2.	클라이언트가 HTTPS 요청을 보냄 → 포트 443
	•	targetPorts.https: http 설정에 따라, 복호화 후 http라는 내부 포트로 트래픽 전달
	•	여기서 http는 실제 서비스가 처리할 비암호화 HTTP 요청을 수신하는 포트


---
## 6. NGINX Ingress Controller vs AWS ALB Ingress Controller

Kubernetes에서 Ingress Controller는 외부 트래픽을 내부 서비스로 전달하는 핵심 컴포넌트입니다.
AWS 환경에서는 대표적으로 두 가지 선택지가 있습니다: NGINX Ingress Controller와 AWS ALB Ingress Controller.
각 컨트롤러는 목적과 특성이 다르기 때문에 상황에 맞는 선택이 중요합니다.

| 항목                         | NGINX Ingress Controller                          | AWS ALB Ingress Controller                         |
|----------------------------|---------------------------------------------------|---------------------------------------------------|
| **설치 방식**                | Helm 또는 YAML로 직접 배포                        | AWS Load Balancer Controller 설치 필요            |
| **Ingress 리소스 지원**      | `Ingress` 리소스 사용                             | `Ingress` 리소스 사용 (주로 ALB IngressClass로 구분) |
| **라우팅 기반**              | NGINX 규칙 기반 (path, host 등)                   | ALB 규칙 기반 (path, host, header 등)             |
| **서비스 노출 방식**         | 내부 Pod + Service → NLB 연결                    | ALB가 직접 각 서비스로 트래픽 전달                |
| **HTTPS 처리**              | NGINX 내에서 SSL 처리 가능                     | ALB에서 직접 SSL 처리                             |
| **Cloud Native 연동성**     | 클라우드 독립적 (AWS 외에서도 사용 가능)           | AWS 전용                                          |
| **IP 고정 가능 여부**        | NLB 설정 시 가능 (Static IP)                     | ALB는 고정 IP 제공 안 됨 (DNS만 제공)             |



 어떤 상황에 어떤 걸 선택해야 하는지

✅ NGINX Ingress Controller 추천 상황
	•	다양한 미들웨어 기능이 필요한 경우 (Rate Limit, Auth, Rewrite 등)
	•	멀티 클라우드나 온프레미스도 고려 중인 환경
	•	세밀한 트래픽 제어가 필요한 복잡한 아키텍처

✅ AWS ALB Ingress Controller 추천 상황
	•	AWS에만 완전히 종속된 인프라를 운영 중인 경우
	•	Ingress 설정만으로 ALB 생성 및 인증서 설정을 자동화하고 싶은 경우
	•	ACM, Cognito 등 AWS 보안 서비스와 통합하고 싶은 경우

---

🧠 결론

유연하고 커스터마이징 가능한 Ingress 환경	-> **NGINX Ingress Controller**

AWS 중심의 자동화된 관리와 쉬운 인증 연동	-> **AWS ALB Ingress Controller**


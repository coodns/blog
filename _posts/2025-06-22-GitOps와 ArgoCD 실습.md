---
layout: post
category: k8s
---
## 1. GitOps란 무엇을 말하는 건가요?

GitOps는 **Git을 단일 진실 공급원(Single Source of Truth)으로 사용하여 인프라와 애플리케이션을 선언적으로 관리하는 운영 방법론**입니다. 간단히 말하면 **'코드로 인프라를 운영하는 방식'**입니다. 우리가 소스코드를 GitHub 같은 곳에 저장하듯이, 인프라 설정도 Git에 저장하고, 이를 기준으로 자동으로 클라우드 환경이 구성되도록 하는 것이죠.

* "우리 앱은 3개의 서버에서 돌아가야 해!" → 이걸 문서로만 남기지 않고, Git에 yaml 같은 파일로 선언합니다.
* Git에 파일을 올리면, 클라우드 시스템이 그걸 보고 "오~ 그럼 3개의 서버 만들게!"라고 반응합니다.


**GitOps의 핵심 원칙:**
- **선언적(Declarative)**: 시스템의 원하는 상태를 코드로 정의
- **버전 관리(Versioned)**: 모든 변경사항이 Git에 기록
- **자동화된 배포(Automated)**: Git 변경사항이 자동으로 클러스터에 반영
- **지속적인 조정(Continuously Reconciled)**: 실제 상태와 원하는 상태를 지속적으로 비교하여 동기화

> ***GitOps의 장점***
>
> - **투명성**: 모든 변경사항이 Git 히스토리에 기록
> - **보안**: Git의 접근 제어와 승인 프로세스 활용
> - **롤백**: Git revert를 통한 쉬운 롤백
> - **감사**: 누가, 언제, 무엇을 변경했는지 추적 가능

---

## 2. CI/CD는 무엇인가요?

CI/CD는 **지속적 통합(Continuous Integration)과 지속적 배포(Continuous Deployment/Delivery)**를 의미합니다.

### CI (Continuous Integration)
개발자가 코드를 자주 병합할 수 있도록 도와주는 자동화를 말합니다.

- **코드 변경사항을 자주 메인 브랜치에 통합**
- **자동화된 빌드와 테스트 실행**
- **코드 품질 검증 및 충돌 조기 발견**

### CD (Continuous Deployment/Delivery)

완성된 코드를 자동으로 서버에 배포해주는 과정입니다. 배포 준비된 코드 혹은 코드들에 대한 변경사항들을 실제 서비스 환경에 배포되도록 해주는 자동화 프로세스들을 말합니다.


---

## 3. GitOps vs CI/CD

| 구분 | 전통적인 CI/CD | GitOps |
|------|---------------|--------|
| **배포 방식** | Push 기반 (CI/CD 도구가 클러스터에 배포) | Pull 기반 (클러스터 내 에이전트가 Git에서 가져옴) |
| **접근 권한** | CI/CD 도구가 클러스터 접근 권한 필요 | 클러스터 외부에서 직접 접근 불필요 |
| **보안** | CI/CD 파이프라인이 클러스터 자격증명 보유 | 클러스터 내부에서만 자격증명 관리 |
| **상태 관리** | 배포 시점의 상태만 관리 | 지속적으로 원하는 상태와 실제 상태 비교 |
| **롤백** | 별도 롤백 파이프라인 필요 | Git revert로 간단한 롤백 |
| **가시성** | 파이프라인 로그 확인 필요 | Git 히스토리로 모든 변경사항 추적 |

**GitOps 워크플로우:**
```
개발자 코드 변경 → Git Push → GitOps 에이전트가 변경사항 감지 → 클러스터에 자동 배포
```

---

## 4. OCI (Open Container Initiative)가 뭘까요?

OCI는 **컨테이너 형식과 런타임에 대한 개방형 업계 표준을 만들기 위한 Linux Foundation 프로젝트**입니다.

**OCI 주요 사양:**
- **Runtime Specification**: 컨테이너 실행 방법 정의
- **Image Specification**: 컨테이너 이미지 형식 정의
- **Distribution Specification**: 컨테이너 이미지 배포 방법 정의

**OCI 호환 레지스트리 예시:**
- Docker Hub
- Amazon ECR
- Google Container Registry
- Harbor
- Quay.io

> ***OCI의 중요성***
>
> OCI 표준을 따르면 **벤더 종속성 없이 다양한 컨테이너 런타임과 레지스트리 간 호환성**을 보장할 수 있습니다.

---

## 5. OCI 기반 레지스트리 vs Helm Chart Repository

| 구분 | 전통적인 Helm Chart Repository | OCI 기반 레지스트리 |
|------|------------------------------|-------------------|
| **저장 방식** | HTTP 서버 기반 (index.yaml 파일) | 컨테이너 이미지와 동일한 방식 |
| **버전 관리** | Chart.yaml의 version 필드 | OCI 태그 시스템 활용 |
| **보안** | 기본 HTTP 인증 또는 별도 구현 | OCI 표준 인증 메커니즘 |
| **저장소 통합** | 별도 Helm 저장소 필요 | 컨테이너 이미지와 동일한 레지스트리 사용 |
| **캐싱** | 제한적 | OCI 레지스트리의 레이어 캐싱 활용 |
| **압축** | .tgz 파일 | OCI 레이어 압축 최적화 |

**OCI 기반 Helm Chart 사용 예시:**
```bash
# OCI 레지스트리에 Helm Chart 푸시
helm push mychart-0.1.0.tgz oci://registry.example.com/charts

# OCI 레지스트리에서 Helm Chart 설치
helm install myapp oci://registry.example.com/charts/mychart --version 0.1.0
```

---

## 6. EKS에서 GitOps를 구현하기 위한 툴링

### 6.1 GitOps 도구들

| 도구 | 설명 | 특징 |
|------|------|------|
| **ArgoCD** | CNCF 졸업 프로젝트 | ✅ 강력한 UI<br>✅ 멀티 클러스터 지원<br>✅ 풍부한 기능 |
| **Flux** | CNCF 인큐베이팅 프로젝트 | ✅ 경량<br>✅ GitOps Toolkit 기반<br>✅ 모듈식 구조 |
| **Tekton** | Kubernetes 네이티브 CI/CD | ✅ CRD 기반<br>✅ 파이프라인 재사용성 |

### 6.2 EKS 통합 고려사항

**AWS 서비스와의 통합:**
- **AWS Load Balancer Controller**: ALB/NLB 자동 생성
- **External DNS**: Route53과 연동한 DNS 자동 관리
- **AWS Secrets Manager**: 시크릿 관리
- **IAM Roles for Service Accounts (IRSA)**: 세밀한 권한 제어

---

## 7. ArgoCD는 무엇인가요? 설치해보세요

ArgoCD는 **Kubernetes를 위한 선언적 GitOps 지속적 배포 도구**입니다.

### 7.1 ArgoCD 설치

```bash
# ArgoCD 네임스페이스 생성
kubectl create namespace argocd

# ArgoCD 설치
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 7.2 ArgoCD 서버 접근 설정

```bash
# ArgoCD 서버를 LoadBalancer로 노출
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

# 또는 포트 포워딩 사용
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### 7.3 초기 관리자 비밀번호 확인

```bash
# ArgoCD 초기 admin 비밀번호 확인
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### 7.4 ArgoCD CLI 설치

```bash
# macOS
brew install argocd

# Linux
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
```

---

## 7.1 ArgoCD를 통한 애플리케이션 배포 실습

### 샘플 애플리케이션 배포

먼저 배포할 애플리케이션의 매니페스트를 Git 저장소에 준비합니다.

**nginx-app.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80

---

apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

### ArgoCD Application 생성
```sh
kubectl creatge argocd-deploy
```

**application.yaml**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/coodns/enhancement
    targetRevision: HEAD
    path: nginx_argo
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd-deploy
  syncPolicy:
    automated:
      prune: true //git에서 삭제시 같이 삭제 시킬 것인지 여부
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

```bash
# Application 배포
kubectl apply -f application.yaml
```

### ArgoCD 동작 원리

**ArgoCD 동작 과정:**

1️⃣ **Git Repository 모니터링**
   - ArgoCD가 지정된 Git 저장소를 주기적으로 폴링
   - 새로운 커밋이나 변경사항 감지

2️⃣ **상태 비교 (Diff)**
   - Git의 원하는 상태와 클러스터의 실제 상태 비교
   - OutOfSync 상태 감지 시 동기화 필요 판단

3️⃣ **자동 동기화 (Sync)**
   - `syncPolicy.automated`가 활성화된 경우 자동 동기화
   - 수동 동기화도 UI나 CLI를 통해 가능

4️⃣ **Self-Healing**
   - 클러스터에서 리소스가 수동으로 변경되거나 삭제된 경우
   - Git의 상태로 자동 복구

5️⃣ **Pruning**
   - Git에서 제거된 리소스를 클러스터에서도 자동 삭제

### ArgoCD UI에서 확인

ArgoCD UI에 접속하면 다음과 같은 정보를 확인할 수 있습니다:

- **애플리케이션 상태**: Healthy, Synced, OutOfSync 등
- **리소스 트리**: 배포된 모든 Kubernetes 리소스의 계층 구조
- **동기화 히스토리**: 배포 이력과 Git 커밋 정보
- **이벤트 로그**: 동기화 과정에서 발생한 이벤트들

---

## 8. CNCF 프로젝트 중 ArgoCD 외 다른 GitOps 도구들

### 8.1 Flux

**Flux v2 (GitOps Toolkit)**
- **특징**: 모듈식 구조, 경량, Kubernetes 네이티브
- **구성 요소**: 
  - Source Controller: Git, Helm, OCI 저장소 관리
  - Kustomize Controller: Kustomize 기반 배포
  - Helm Controller: Helm Chart 배포
  - Notification Controller: 알림 및 웹훅


### 8.2 Tekton

**Tekton Pipelines**
- **특징**: Kubernetes 네이티브 CI/CD, CRD 기반
- **구성 요소**:
  - Task: 개별 작업 단위
  - Pipeline: Task들의 조합
  - PipelineRun: Pipeline 실행 인스턴스


### 8.4 비교 요약

| 도구  | 주요 특징 | 적합한 사용 사례 |
|------|-----------|-----------|-----------------|
| **ArgoCD** | 강력한 UI, 멀티클러스터 | 복잡한 환경, UI 중심 운영 |
| **FluxCD**  | 경량, 모듈식 | 단순한 환경, 자동화 중심 |
| **Tekton**  | Kubernetes 네이티브 | CI/CD 파이프라인 구축 |

각 도구는 고유한 장단점이 있으므로, **팀의 요구사항과 기존 인프라를 고려하여 선택**하는 것이 중요합니다.

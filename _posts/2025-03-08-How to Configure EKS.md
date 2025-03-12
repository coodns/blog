---
layout: post
category: k8s
---

## 1.  일반적인 Kubernetes 와 EKS 의 kubeconfig 파일 차이점

![image](https://github.com/user-attachments/assets/567c3685-e558-4eea-b724-0692507dd594)

기본적인 kubeconfig 파일은 보통 아래와 같은 요소를 포함합니다
```
apiVersion: v1
kind: Config
clusters:
- name: my-cluster
  cluster:
    server: https://my-cluster-api-server
    certificate-authority-data: BASE64_ENCODED_CA
users:
- name: my-user
  user:
    client-certificate-data: BASE64_ENCODED_CERT
    client-key-data: BASE64_ENCODED_KEY
contexts:
- name: my-context
  context:
    cluster: my-cluster
    user: my-user
current-context: my-context
```


일반적인 Kubernetes 와 EKS 의 kube config의 주요 차이점으로는 아래와 같습니다.


API 엔드포인트: EKS는 관리형 Kubernetes 클러스터이므로, API 서버 엔드포인트가 AWS에서 제공됩니다.
IAM 기반 인증: kubeconfig에서 직접 토큰을 저장하는 것이 아니라 aws eks get-token 명령어를 통해 동적으로 인증을 수행합니다.
CA 인증서 포함: AWS 제공 인증서가 포함되어 있으며, 자체적으로 변경이 어렵습니다. 즉, 일반 Kubernetes에서는 직접 관리하는 CA 및 인증서를 사용하지만, EKS에서는 IAM 기반 인증을 사용하여 자동으로 토큰을 가져옵니다.


| 항목        | 일반 Kubernetes      | EKS                   |
|------------|--------------------|----------------------|
| 인증 방식   | x509 인증서         | IAM을 통한 인증       |
| API 엔드포인트 | 직접 설정           | AWS가 제공            |
| kubeconfig 내 토큰 | 직접 포함 가능   | `aws eks get-token` 사용 |


## 2. EKS Authentication & Authorization

### 2.1 Role Based Access Control (RBAC)

EKS의 인증과 인가는 IAM과 Kubernetes RBAC(Role-Based Access Control)를 조합하여 이루어집니다.
Role-Based Access Control (RBAC)은 사용자, 그룹 또는 서비스 계정(SA)에 클러스터 내의 리소스에 대한 접근을 제어하는 방법입니다.


**인증 (Authentication)**

aws eks get-token 명령을 사용하여 AWS IAM을 통해 인증 토큰을 생성합니다.
AWS IAM을 통해 aws-auth ConfigMap에 등록된 사용자인지 확인합니다.
IAM 역할과 사용자 정책에 따라 API 서버 접근이 허용됩니다.


**인가 (Authorization)**

aws-auth ConfigMap을 통해 IAM 역할과 Kubernetes RBAC를 연결합니다.
RBAC(Role-Based Access Control)를 활용하여 권한을 설정합니다.
예를 들어, 특정 IAM 사용자가 클러스터의 admin 또는 readonly 권한을 갖도록 설정할 수 있습니다.

![Image](https://github.com/user-attachments/assets/a53d5ebb-0273-48cb-a6c9-8c0fa5732015)


### 2.2 다른 사용자에게 클러스터 접근 권한 부여하기

EKS 클러스터에 다른 사용자를 추가하려면 aws-auth ConfigMap을 수정해야 합니다.

```yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: arn:aws:iam::123456789012:role/EKSAdminRole
      username: admin
      groups:
        - system:masters
  mapUsers: |
    - userarn: arn:aws:iam::123456789012:user/developer
      username: developer
      groups:
        - system:basic-user
        - system:view
```
- mapRoles: IAM 역할을 Kubernetes 그룹과 연결합니다.
- mapUsers: 특정 IAM 사용자를 Kubernetes 사용자로 매핑합니다.
- groups: RBAC를 사용하여 사용자에게 특정 권한을 부여할 수 있습니다.

이렇게 설정하면, 해당 IAM 사용자 또는 역할을 가진 계정이 kubectl을 통해 클러스터에 접근할 수 있습니다.

혹은 eksctl 을 사용할경우 아래와 같은 명령어를 통해 진행 할 수 있습니다.


```eksctl
eksctl create iamidentitymapping --cluster my-cluster --region=region-code \
    --arn arn:aws:iam::111122223333:role/my-role --username admin --group binding-name \
    --no-duplicate-arns
```

## 3. Pod는 배포 시 어디로 가나요?
Pod가 배포될 때, Kubernetes의 스케줄러가 해당 Pod를 어느 노드(Node)에 배치할지를 결정합니다. 기본적으로 Kubernetes는 여러 요소를 고려하여 최적의 노드를 선택합니다.

### 3.1 배포 방법 종류

#### Pod
#### Deployment
#### DaemonSet
#### Stateful
#### Job
#### Cronjob


### 3.2 내가 원하는 노드에 Pod를 띄우는 방법

1. nodeSelector
2. nodeAffinity

이외에도 Taints & Tolerations, PodTopologySpreadConstraints 가 있습니다.

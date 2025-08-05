---
layout: post
category: k8s
---

# Infrastructure as Code

Infrastructure as Code(IaC)는 일반적으로 수동으로 각각의 인프라 리소스를 생성하는 것이 아니라, 특정 언어나 양식에 맞춘 **코드로 인프라를 관리 및 프로비저닝**하는 방식을 의미합니다.

예를 들어 AWS에서 웹서버, 데이터베이스 등을 포함하는 웹 애플리케이션 환경을 구축한다고 가정해보겠습니다. 이를 진행하기 위해서는 AWS VPC, EC2, RDS 등 서비스들에 대해 직접 콘솔에 액세스하거나 CLI 커맨드를 통해 각각 생성해야 합니다. 하지만 이는 시간이 오래 걸리고, 서비스 간의 호환성이나 설정의 일관성을 유지하기 어렵습니다.

반면 **코드형 인프라**를 사용할 경우, 인프라 구축 및 관리를 코드로 정의하고 관리할 수 있어 반복적이고 수동적인 작업들을 자동화하고, 인프라의 재현성과 안정성을 보장할 수 있습니다.

---

## IAC 실행 방식

- ### 선언형(Declarative)
  사용자는 "무엇을 만들 것인가(What to build)"를 정의합니다. 테라폼은 현재 상태와 원하는 상태를 비교하여 필요한 작업만 수행합니다.
  - 예: `resource "aws_instance" "web" { ... }` 라고 선언하면, 해당 인스턴스를 만들거나, 존재한다면 변경하지 않습니다.

- ### 명령형(Imperative)
  사용자가 "어떻게 만들 것인가(How to build)"를 직접 정의합니다. 단계별로 순차적 명령을 실행하는 방식이며, Ansible, Bash Script 등이 해당됩니다.
  - 예: `aws ec2 run-instances ...` 명령으로 직접 실행
- ### 절차형
  프로그래밍 언어를 이용해서 직접 순차적으로 인프라를 생성하도록 코드를 작성하는 방법이다. 선언형에 비해 프로그래밍 언어적 특성을 살려 더 강력한 일들을 할 수 있으나, 실제 적용된 결과를 가늠하기 어렵고, 코드를 읽기에 직관적이지 않다.
  - 예: `aws cdk`,`plumi` 등의 도구

Terraform은 **선언형(Declarative)** 접근 방식을 따릅니다
Terraform은 선언형 방식이기 때문에 **상태(state)** 를 기반으로 계획(plan) → 실행(apply) → 반영을 수행하며, 반복 가능하고 재현 가능한 인프라 구성이 가능합니다.

## 구성 조정 도구(Configuration Orchestration Tool) & 구성 관리 도구(Configuration Management Tool)
 
 - Terraform과 AWS CloudFormation과 같은 구성 조정 도구(Configuration Orchestration Tool)는 서버 및 기타 인프라의 구축을 자동화 하도록 설계됨.
 - Chef, Puppet과 같은 구성 관리 도구(Configuration Orchestration Tool)는 이미 프로비저닝된 이 인프라의 소프트웨어와 시스템을 구성하고 관리하는데 도움이 됨

두 형식 모두 인프라를 생성하고 자동화 한다는 동일한 목적을 가지고 있지만, 각 형식별 담당하는 역할과 범위가 다릅니다.

ex) Terraform (구성 조정 도구)
→ “EC2 인스턴스 3개, VPC, ALB, RDS 만들어줘”

ex) Chef (구성 관리 도구)
→ “이 EC2 인스턴스들에 NGINX 설치하고, 설정 파일 복사하고, 서비스 시작해줘”

## 비교

| 항목                    | AWS CDK                                              | Terraform                                | Ansible                             |
|-------------------------|------------------------------------------------------|-------------------------------------------|-------------------------------------|
| **도구 유형**           | 구성 조정 도구 (Orchestration)                             | 구성 조정 도구 (Orchestration)            | 구성 관리 도구 (Configuration Management) |
| **실행 방식**           | 절차형 + 선언형 혼합 (리소스의 최종 상태를 정의하고 CloudFormation으로 변환됨) | 선언형                                     | 명령형 (또는 선언형 YAML 기반)                |
| **사용 언어**           | Python, TypeScript, Java 등                           | HCL (HashiCorp Configuration Language)     | YAML (Playbook), Jinja2 템플릿         |
| **상태 관리**           | 내부적으로 CloudFormation 사용                              | 상태 파일(`terraform.tfstate`) 기반       | 상태 저장 없음 (푸시 기반)                    |
| **적합한 용도**         | AWS 전용 IaC, CI/CD 자동화 스크립트 포함                        | 멀티클라우드 인프라 배포, 재현 가능한 구조 설계 | 서버 설정, 패키지 설치, 환경 초기화 등             |
| **장점**                | - 프로그래밍 언어 사용 가능<br>- AWS 서비스와 자연스러운 통합              | - 멀티 클라우드 지원<br>- 커뮤니티와 모듈이 풍부 | - 에이전트 설치 필요 없음<br>- 서버 내부 세팅에 최적   |
| **단점**                | - AWS 전용<br>- 코드 추상화로 결과 직관성 떨어짐                     | - HCL 문법 한계<br>- 복잡한 로직 구현 어려움 | - 선언형 불완전<br>- 상태 추적 어려움       [    |

# Terraform

Terraform 은 HashiCorp에서 만든 오픈소스 **인프라 자동화 도구**로, 다양한 클라우드 서비스(AWS, GCP, Azure 등)를 추상화하여 일관된 방식으로 인프라를 코드로 정의할 수 있게 해줍니다.
오픈 소스 기반의 구성 조정 도구로써 (Configuration Orchestration Tool)로 클라우드/온프라미스에 구애 받지 않고 사용할 수 있는 인프라 프로비저닝 도구입니다.
 
 - 하시코프사(Hashicorp)에서 개발하였으며, Go로 작성되었음
 - 퍼블릭 및 프라이빗 클라우드 인프라 프로비저닝을 지원
 - 다른 구성 조정 도구와 달리 Terraform은 상태 파일이라는 개념을 사용하여 인프라의 상태를 유지 가능
 - 자체 HCL(Hashicorp Configuration Language)라는 DSL(Domain Specific Language) 사용
 - HCL은 JSON과 호환되며 배포할 인프라 자원을 설명하는 구성 파일을 만드는 데 사용
 - Custom 기능 개발을 위해 Terrform Plug-in을 사용하여 플랫폼에 고급 기능 추가


*DSL(Domain Specific Language) : 도메인(산업, 분야등 특정 영역)에 특화된 언어*

## Components

Terraform은 다음과 같은 주요 구성요소로 이루어져 있습니다:

- **Provider**: AWS, GCP 등의 클라우드 API와 통신하여 리소스를 생성/삭제하는 플러그인
- **Resource**: 실제 생성할 인프라 리소스를 정의 (예: `aws_instance`, `aws_s3_bucket`)
- **Data**: 외부 데이터를 가져와서 참조할 때 사용
- **Variable**: 입력값을 받아 코드에 유연성을 제공
- **Output**: 생성된 리소스의 값을 출력하여 다른 모듈에서 사용 가능
- **Module**: 반복되는 코드 블럭을 재사용 가능하도록 묶은 구성 단위
- **State File**: 현재 인프라 상태를 저장하는 파일 (`terraform.tfstate`)
- **Backend**: 상태 파일을 저장하는 위치 (S3, local 등)
- **Locking**: 동시 작업을 방지하기 위한 락 메커니즘 (예: DynamoDB 사용)

## workflow

![img](https://media.geeksforgeeks.org/wp-content/uploads/20230606114940/Terraform-flow-chartr-(2).webp)


# CDK 
AWS 에서 개발한 프로그래밍 언어를 사용하여 인프라를 코드로 정의하는 오픈 소스 소프트웨어 개발 프레임워크입니다. 
TypeScript, Python, Java, C# 등 다양한 언어를 지원합니다.

## Components

- **Construct** : CDK에서 재사용 가능한 컴포넌트를 말합니다. 이것은 하나 이상의 AWS CloudFormation Resources 및 설정이 캡슐화 되어있으며, 추상화 단계에 따라 Level 1~3로 분류합니다.
![testest](https://devocean.sk.com/editorImg/2024/3/24/c094b98eb5501446a82ec024633407c09853537c194c8a7dd5259c6fb8f3203e)
  - Level 1: CloudFormation에서 사용할 수 있는 모든 AWS 리소스를 직접 나타냅니다. 
  - Level 2: AWS 리소스를 나타내고, 세부정보를 추상화해서 제공합니다. 
  - Level 3: 여러 리소스를 함께 연결하며 참조 아키텍처 또는 설계 패턴을 나타냅니다.


- **Stack** : Stack은 CDK의 배포 단위입니다. Stack 범위 내에서 직접 또는 간접적으로 정의된 모든 AWS 리소스는 하나의 묶음으로 프로비저닝됩니다. Stack은 AWS CloudFormation 스택을 통해 구현되므로 동일한 제한 사항이 있습니다.
- **App** : 하나 이상의 스택을 위한 컨테이너로, 각 스택의 Scope을 정의합니다. 단일 App 내의 Stack은 서로의 자원과 속성을 쉽게 참조 할 수 있습니다. CDK는 Stack간의 종속성을 유추하여 올바른 순서로 배포 할 수 있습니다.

## Workflow

![imge](https://devocean.sk.com/editorImg/2024/3/24/fcb5f3466f8cc21a509bd0f0889c9336e2aa3a726025aea7569c815e11e05173)

1. 애플리케이션을 만들고 실행합니다.
2. 정의한 애플리케이션 모델을 조사하고 합성(Synthesize)을 합니다.
3. AWS CDK에 의해 생성된 AWS CloudFormation 템플릿을 작성하고 배포합니다.


# AWS EKS 대표 모범사례 몇가지

| 모범 사례                                       | 이유                                                                                                                                                 | 
|------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| **1. Control Plane 접근 최소화**              | EKS API 서버(제어 플레인)의 **퍼블릭 노출을 줄이고**, 특정 CIDR 또는 VPC 내부에서만 접근하도록 구성하면 **외부 공격 면을 대폭 줄일 수 있음**.                            | 
| **2. IAM Roles for Service Accounts (IRSA)** 사용 | 각 Pod에 IAM 권한을 할당하면, **Node에 붙은 과도한 IAM 권한을 제거**하고 **서비스별 최소 권한 원칙(Least Privilege)**을 준수할 수 있음.                                   | 
| **3. Control Plane 모니터링 활성화**          | API 서버, etcd, scheduler 등의 **제어 플레인 구성요소 상태를 지속적으로 확인**하면, 장애나 병목을 **사전에 감지**하고 빠르게 대응할 수 있음.                              | 
| **4. TLS/HTTPS 및 내부 보안 통신 구성**        | ALB, Ingress, 서비스 간 통신에 **TLS 암호화를 적용**하면 데이터 도청/변조를 막을 수 있고, 내부적으로는 **mTLS, CNI 정책** 등을 활용해 **Pod 간 통신 보안도 강화 가능**.       |
| **5. IaC 기반 인프라 관리**                    | Terraform, AWS CDK 등을 사용하면 **리소스 변경 이력 추적, 자동화, 검토 및 롤백**이 가능해지며, 수동 운영에서 오는 실수와 불일치를 방지할 수 있음.                          | 
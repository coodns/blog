---
layout: post
category: k8s
---

## EKS Logical On Boarding
### 1. kubernets는 정확히 무엇인가요?
쿠버네티스는 컨테이너를 쉽고 빠르게 배포 및 확장하고 관리를 자동화 해주는 오픈소스 플랫 폼입니다. 



### 2. kubernetes는 무엇으로 이루어져 잇나요??
￼	쿠버네티스를 배포하게 되면, 쿠버네티스 클러스터 형태로 배포가 됩니다. 
	클러스터 노드라고 하는 워커 머신들의 집합을 의미하며 모든 클러스터는 최소 하나의 워커 노드를 가지게 됩니다.  
	</br>
	워커노드는 pod 를 호스팅 하는 목적을 두고있으며, ControlPlane 은 Worker Node 와 클러스터 내 Pod들을 관리합니다.


### 3. Control Plane ( Master Node ) Components  

Control Plane ( Master Node ) 구성요소는 클러스터에 관한 전반적인 결정들을 수행하며 이벤트를 감지하고 반응 하도록 하는 요소들입니다.
예를 들어 Pod 스케줄링 을 하여 파드들이 생성되는것을 일정화 시킬 수 있으며, Deployment 의 Replicas 필드에 대한 요구 조건이 충족되지 않을 경우 새로운 파드를 구동시키는 것이 있습니다. 
</br></br>
위와같이, Control Plane ( Master Node ) 구성요소들은 클러스터 내 어떠한 머신에서든지 동작이 가능하며, 대부분 Control Plane ( Master Node ) 을 동일한 컴퓨팅 머신상에 구성시키고, 사용자 컨테이너는 해당 컴퓨팅 머신에 동작시키지 않습니다.
아래 Control Plane ( Master Node ) 의 구성 요소들에 대해 살펴 보겠습니다. 
</br></br>
	![Image](https://github.com/user-attachments/assets/bb9d1dd2-c72f-4719-8df1-9bf214486441)

#### 3.1 Kube-apiserver
API 서버는 쿠버네티스 API를 노출하는 쿠버네티스 컨트롤 플레인 컴포넌트입니다. 

- kubernetes을 클라우드에 배포하는거랑 온프레미스에 배포하는거랑 뭐가 다른가요?


### Node Components
노드 구요소들의 경우 동작 중인 파드를 유지시키고
#### 4.1 kubelet 
#### 4.2 kubenetes controlplan과 dataplan의 차이
- 위 둘은 어떻게 통신하나요?
- 둘 중 어디다가 설치할까요? 어느떄? 어느상황에?
- 쿠버네티스는 결국 어느 상황에 적합하나요?
- pod와 컨테이너의 차이를 설명하세요
- kuberentes는 컨테이너를 어떻게 실행하나요? 누가 실행하나요?

1. EKS의 kubeconfig는 어떤 구조인가요
- 그냥 kubernets를 사용햇을떄 kubeconfig와 eks로 구성된 클러스터와 연동 시 kubeconfig는 어떻게 다른가요?
- eks에서의 인증/인가는 어떻게 이루어지나요?
- 만든사람 이외에 유저에게 클러스터 권한을 어떻게 주나요?
- 참고: 아래 명령어를 통해 eks는 클러스터의 대한 kubeconfig을 가져올 수 있습니다

		aws eks update-kubeconfig

1. Pod는 배포 시 어디로 가냐요?
- nginx pod단 하나를 띄웟을떄 어디로 갈까요?
- deployment로 뿌리면 어떻게 달라지나요?
    - 배포 방법이 위 두개 뿐인가요?
- 배포 방법은 각각 어떨데 사용하나요?
- 내가 원하는 곳에 띄우는 방법은 없나요?

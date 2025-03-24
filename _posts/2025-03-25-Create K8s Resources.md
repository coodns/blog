---
layout: post
category: k8s
---

## 1. configure nginx pod

## 1.Scailing

2. 쿠버네티스 위 pod들은 어떻게 스케일링 되나요?

3. 스케일링 방법을 알려주세요

4. 각 방법들은 어떤경우 설정하나요?

5. 어느 경우에 적합하나요?

6. 무조건 해야하나요?

7. 제한 사항은요?

- 스케일링은 부하량이 증가 또는 감소함에 따라 더 많은 자원 할당이 필요하거나, 낭비되고 있는 자원을 최소화 해야 할 때 사용합니다.
- Scaling은 수평/수직이 있으며, 수직은 Pod의 Resource 스펙을 높이거나 낮추는 방법이고, 수직은 Pod 개수를 늘리거나 줄이는 방법입니다.

- 수직은 주로 크기 조절이 불가능한 DaemonSet에 어울리는 방식이며, 수평은 Deployment 및 Statefulset에 어울리는 방식입니다.

- 수평 스케일링은 HPA(Horizontal Pod AutoScaler)를 통해 구현할 수 있으며, 일반적으로 CPU, Memory 및 커스텀 지표를 통해 스케일링을 진행합니다.
- CPU와 같은 메트릭을 기반으로 조건을 설정하는 경우 Metrics Server와 같은 Addon이 필요하며 별도로 구성해야 합니다.

- Metrics Server 및 HPA를 구성하면 HPA에 설정된 메트릭 조건을 기반으로 Pod가 Scale in/out을 진행합니다.
- HPA Controller가 메트릭을 주시하고, 스케일링 조건에 부합하면 Deployment 또는 StatefulSet의 replicaSet의 크기를 업데이트하여 Desire을 수정하고, Current를 맞추는 방식으로 동작합니다.

- 스케일링을 꼭 구성해야 하는 것은 아니지만,
유동적인 트래픽 흐름에 대응하고 낭비되는 리소스를 최소화 하기 위해 스케일링 구성이 권장됩니다.

- 앞에서 설명한대로 DaemonSet은 Scaling in/out 방식을 사용할 수 없기 때문에 리소스에 맞게 적절한 스케일링 방식을 가져가는 것이 좋으며, CPU와 같은 메트릭을 조건으로 설정하기 위해선 별도로 MetricsServer와 같은 애드온 구성이 필요합니다.
- 삭제 타임스탬프가 설정되어 있거나 실패한 Pod는 포함되지 않으며, Ready 되지 않은 Pod는 일시적으로 제외됩니다.
	- Ready가 되지 않은 Pod는 ```--horizontal-pod-autoscaler-initial-readiness-delay```(기본값 30초) 및 ```--horizontal-pod-autoscaler-cpu-initialization-period```(기본값 5분) 설정을 통해 Pod가 준비되는 시점을 판단합니다.
- 준비되지 않은 포드와 누락된 메트릭을 고려하지 않는 경우 원하는 메트릭의 0%를 사용한다고 보수적으로 가정하여 확장 규모를 더욱 줄이며, 측정값이 없는 Pod는 Scale In 시 100% 사용한 것으로 간주하고, Scale Out 시 0% 사용한 것으로 간주합니다.

## 2. Exposing
1. pod 끼리 통신은 어떻게 하나요? 또는 어떻게 호출하나요
2. 어떻게 그게 가능하죠?
3. 그럼 이제. pod을 인터넷에서 볼수 있게 해봅시다 nginx pod을 배포하고 저희가 어디서든 볼수 있게 해주세요
4. 어떤 리소스로 위 사항이 가능하나요?
5. ingress-controller가 무엇인지 설명하세요
6. 어떻게 pod와 연결하는지도 설명하세요
7. pod의 헬스체크는 어떻게 이루어지나요?
8. 헬스체크가 실패하면 트래픽이 더이상 가지 않나요?
9. 위를 이루기 위해 어떤 설정이 필요한가요

nginx pod 배포 실습 nginx pod, ingress pod, pod health check 구성이 포함되어야함.

## 3. Storage
1. kubernetes에서 볼륨에 관련된 리소스들은 무엇이 있나요?
2. ebs-csi-driver가 뭔지 찾고 구성하세요
3. 스토리지 클래스가 무엇인가요?
4. 스토리지 클래스를 사용하여 Deployments Pod의 볼륨을 붙여보세요
5. deployment를 스케일링하세요
6. 실패하였다면 이유를 알아내세요
7. Pod가 다시 볼륨을 사용할 수 있도록 트러블슈팅하세요
8. Deployment와 StatefulSet의 차이를 공부하세요
9. 두개는 어느상황에 사용해야하나요??


## 2. 실습 
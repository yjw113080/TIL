
맨날 밋업에 가서 귀동냥만 하다가 처음으로 내가 발표를 해봤다.
cloud jam 에서 같이 공부했던 내용을 정리하고 내가 조금 더 심화 학습했던 내용을 정리해서 공유했다.
생각보다 GDG 에서 많은 분들이 오셔서 당황했지만 따뜻하고 격려해주시는 분위기 속에서 편하게 발표할 수 있었다.
공유했던 내용은 다음과 같다.

## Docker & Kubernetes Introduction
1. Docker
인프라 운영에 있는 사람들이 개발자들과 부딪힐 때 심심지 않게 듣는 말은 "로컬에서는 잘 됐는데요.."이다.
Docker가 들어오면서, 서로 다른 실행 환경에서 lib와 bin 을 모두 말아서 옮겨다닐 수 있기 때문에 컨테이너 기술이 각광 받고 있다.
그렇지만 컨테이너에 대한 운영 관리, 네트워크 관리, 보안 관리 등에 대한 워크로드와 리스크가 있다.
예를 들면, 삼중화된 컨테이너가 하나 죽는다면? 프로세스는 정말 툭하면 죽는데 어떻게 그 때마다 다시 삼중화 구조를 살리지? IP는 대체 누가 어디서 무슨 로직으로 주는 거지? 인증서는 어떻게 심어줘야 하지? 이런 것들의 문제다.

2. Kubernetes
이러한 이유로 Kubernetes와 같은 Orchestration 툴이 등장했다.
개중 k8s가 각광 받는 이유는 3가지로 추려볼 수 있다.
하나, CNCF (Cloud Native Computing Foundation) 의 전폭적인 지원을 받되 코어를 오픈해, 벤더로 들어가더라도 k8s를 이해한다면 vendor lock-in을 피할 수 있다.
둘, 각 기능마다 생성되는 컨테이너들을 어떻게 하나로 묶어서 single application 으로 기능하게 할 것인가에 대한 해결을 pod로 제공한다. 깔끔하고 세련된 추상화 레벨이 단연 최고.
셋, 신경 쓸 필요도 없이 간편하고 신속하게 이루어지는 scaling, load-balancing 역시 인기 요인 중 하나다.

kubernetes 가 제공하는 기본적인 아키텍처 단위는 다음과 같다.

![](https://images.ctfassets.net/hspc7zpa5cvq/7tLkz85tYIUYuai4suAkm2/a8b4437e6e97bbabc6c98917fc130077/Kubernetes-Relationships.png)
Node:  Pod를 실행하기 위한 실제 컴퓨팅 자원 (VM, 물리서버)
Pod: Deployment 유닛(==컨테이너의 집합). 이 안에서는 하나의 IP 주소를 공유하고 localhost를 통해 서로 접근할 수 있음. 볼륨도 공유.
ReplicaSet: pod의 복제 수(이중화/삼중화…). Deployment를 통해 이를 통제하는 것이 권고됨.
Deployment:  Pod와 ReplicaSet 배포/업데이트를 위한 컨트롤러.
Service: pod의 논리적 묶음과 여기에 접근할 수 있는 정책을 정의한 추상화 레벨. Pod replicas에 IP를 할당하여 외부 통신이 가능하게 한다.



## In-depth Discussion
1. Network
IP 통신을 대체 어떻게 하는 걸까??
Kubernetes 클러스터가 실행되는 모든 노드에는 kube-proxy가 함께 실행된다. Kube-proxy는 Service의 Virtual IP를 지정하는 역할을 수행.
단, 외부 IP 지정은 k8s가 아닌 클러스터 관리자가 설정한다.
이 proxy가 어떻게 작동할 건지는 다음과 같은 세 가지(proxy-mode)로 나뉜다.

A. proxy-mode: userspace
노드 로컬에 컨테이너가 직접 포트 오픈
SesisonAffinity에 따라 어떤 backend pod가 사용될 지 알 수 있음.
Iptables 규칙에 따라 서비스 클러스터 IP로 오는 트래픽을 확인한 뒤 백엔드 pod랑 연결. (RoundRobin)

B. proxy-mode: iptables
Iptables 규칙에 따라 서비스 클러스터 IP로 오는 트래픽을 확인한 뒤 백엔드 pod랑 연결.
백엔드 pod는 랜덤으로 선택됨.
Userspace와 kernelspace를 왔다갔다 할 수 없어 신뢰성이 더 높음.
단, 동작하지 않는 pod 가 있는 경우 자동으로 다른 pod에 요청하지 않고 일단 대기 상태로 넘어간다.

C. proxy-mode: ipvs
1.9 beta
Netfliter라는 hook function 을 이용. Hash table 기반이라 트래픽 처리가 매우 빠름.
Load balancing option도 다양.



2. Stateful Apps
일단 최선은 stateless 하게 가는 것. 데이터를 어떻게 들고 있을 것이냐가 배포할 때마다 신경써주어야 해서 되도록이면 브라우저에서 처리하도록 하는 게 좋다.
그렇지만 그럼에도 불구하고 세션이 필요하다면 `service.spec.sessionAffinity : ClientIP`를 사용한다.
이 설정을 넣어주고 나면, 클라이언트의 IP를 확인해서 하나의 백엔드 컨테이너와 통신하게 해준다.

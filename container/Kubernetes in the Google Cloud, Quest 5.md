
## Continuous Delivery with Jenkins in Kubernetes Engine

1. Jenkins 개념
![](https://gcpstaging-qwiklab-website-prod.s3.amazonaws.com/bundles/assets/d5bfbd0f6d109df463005f1ceb195799ec68b7b4c370eb18cec470a7f1f812b1.png)
Jenkins 는 공용 레파지토리에서 코드를 계속해서 통합할 때 사용하는 자동화 서버다. Continuous Delivery 파이프라인을 구성할 때, 쿠버네티스 엔진에 Jenkins 를 구축하는 게 일반적인 VM을 이용하는 것보다 더 좋다. why is that?

Kubernetes Engine 은 구글의 글로벌 로드 밸런서를 내장하고 있다. 그래서 자동 웹 트래픽 라우팅을 이용할 수 있다. 로드 밸런서가 SSL termination 을 처리해주며, 구글의 백본 네트워크를 통해 공용 IP 주소를 할당해준다. 이 로드 밸런서를 통해 사용자들이 언제나 가장 빠른 패스를 통해 어플리케이션 인스턴스에 접근할 수 있도록 한다. What about ICP??


2. Jenkins 인스턴스 생성을 위한 Helm 설치

`kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)`
사용자를 클러스터의 어드민으로 등록한다. 이 과정을 거쳐야 젠킨스에 관리자로 접근할 수 있음!

Tiller, Helm의 서버 쪽인 Tiller 에도 어드민 롤을 추가해준다. 
* Tiller 란, Helm 의 클러스터 내부 컴포넌트. K8S API 서버와 다이렉트로 통신하여 K8S 자원을 설치/업그레이드/쿼리/삭제하도록 한다. 릴리즈 버전에 대한 객체도 보관한다. 
```kubectl create serviceaccount tiller --namespace kube-system
kubectl create clusterrolebinding tiller-admin-binding --clusterrole=cluster-admin --serviceaccount=kube-system:tiller```


3. Jenkins 인스턴스 생성
Helm 의 Chart 레포지토리에서 젠킨스를 끌어와 설치한다. 
`./helm install -n cd stable/jenkins -f jenkins/values.yaml --version 0.16.6 --wait`
설치가 완료되면 Jenkins 인스턴스는 default 네임스페이스에 생성된다.

`kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &`
포트를 포워딩해 Jenkins UI에 접근할 수 있도록 한다.

`printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo`
Jenkins 가 생성될 때 자동으로 admin 계정에 대한 패스워드를 생성한다. 위 명령어를 통해 패스워드를 확인한다.

4. Application Deployment
![](https://gcpstaging-qwiklab-website-prod.s3.amazonaws.com/bundles/assets/3f54f9241596a6b03888b7ff079f8ee9ab3ba2d2c4db960175ee7b8336704b3e.png)
`kubectl create ns production` 네임스페이스 추가해주고
`kubectl apply -f k8s/production -n production` k8s/production 폴더 아래 있는 yaml 파일을 모두 실행한다.

네임스페이스 값을 주면 기존에 디플로이 했던 것과 격리하여 별도로 조회해야 한다. 모든 명령에 `-n XXX` 를 준다고 생각하면 됨.

5. 버전 관리 등록
Jenkins 에 연결해줄 git repository 를 `git init`, `git remote add` 를 통해 설정해준다.

6. Jenkins 파이프라인 설정
6.1. Credential 등록
6.2. Jenkins 트래킹 잡 등록
`Add New Item` 으로 트래킹 할 프로젝트를 추가한다.
개발용 브랜치를 별도로 사용한다는 가정 하에 MultiBranch Pipeline 을 선택한다.

* Jenkins 에 대해서는 심화학습 필요!!!

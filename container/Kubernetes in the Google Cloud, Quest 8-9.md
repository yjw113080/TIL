
## Setting up a Private Kubernetes Cluster

프라이빗 클러스터를 생성할 때에는, k8s 마스터 컴포넌트가 올라갈 노드 VM에 /28 CIDR을 설정한 뒤 IP alias를 주어야 한다.

* CIDR
인터넷 프로토콜은 A Class 에서 D Class 까지 4개의 클래스 내에서 IP 주소를 정의하게 됩니다.
클래스는 각각 32Bit 인터넷 주소형식의 한 부분을 네트웍 주소로 할당하게 되며, 남은 부분은 해당 주소에 의해 지정된 네트웍 내에 잇는 호스트에 할당하게 됩니다

`gcloud compute networks subnets describe [섭넷아이디]` 로 클러스터 생성할 때 같이 생성된 subnet을 확인한다.
```
creationTimestamp: '2019-01-17T18:40:05.608-08:00'
description: auto-created subnetwork for cluster "private-cluster"
...
privateIpGoogleAccess: true
region: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-81ab5c597ae09d8e/regions/us-central1
secondaryIpRanges:
- ipCidrRange: 10.33.0.0/20
  rangeName: gke-private-cluster-services-834b439d
- ipCidrRange: 10.36.0.0/14
  rangeName: gke-private-cluster-pods-834b439d
...
```
* `privateIpGoogleAccess: true` 프라이빗 환경이지만 Google API와는 통신할 수 있도록 설정
* `secondaryIpRanges` 해당 섭넷 아래 서비스와 pod에 대한 영역을 나눈다.


* Enabling master authorized networks
At this point, the only IP addresses that have access to the master are the addresses in these ranges.
To provide additional access to the master, you must authorize selected address ranges.
```
gcloud container clusters update private-cluster \
    --enable-master-authorized-networks \
    --master-authorized-networks [MY_EXTERNAL_RANGE]
```



## Helm Package Manager
Helm is a Kubernetes package manager currently maintained by the Cloud Native Computing Foundation (CNCF), which is a cloud industry consortium composed of Google, Microsoft, and Bitnami amongst other companies. Helm has become quite popular with cloud developers largely because it simplifies Kubernetes application management, the roll out of updates, and options to share applications.

1. Helm을 이용할 클러스터를 먼저 만든다.

2. Helm 설치

3. Helm 시작 및 Tiller 설정
3.1. Tiller 서비스 계정 만들기 + cluster-admin 롤 주기
```kubectl -n kube-system create sa tiller
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller```

3.2. 설치
`helm init --service-account tiller`

4. Chart 설치
Helm 에서 받아갈 수 있는 이미지를 chart 라고 하는 듯.
`helm install stable/mysql` 테스트로 mysql 이미지를 받아본다.
산뜻하게 설치가 완료되고 나면 초기 시작을 위한 가이드가 콘솔에 뿌려진다.
`kubectl get secret --namespace default hopping-cardinal-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo`
루트 패스워드를 받는 방법을 알려주는 게 편리하군.



## NGINX Ingress Controller on Google Kubernetes Engine
쿠버네티스에는 `Ingress`라는 컴포넌트가 있어서, 외부 사용자들과 클라이언트 앱이 HTTP 서비스에 접근할 수 있게 해준다.
Ingress는 Resource와 Controller 로 이루어짐.
- Ingress Resource 는 서비스에 도달하기 위한 인바운드 트래픽에 대한 규칙이다. 특정 호스트네임/path 이 쿠버네티스의 특정 서비스로 접근할 수 있도록 해주는 L7 레이어의 룰.
- Ingress Controller 는 Resouce 에서 정의한 규칙에 기반하여 HTTP, L7 로드밸런서를 통해 실제 작동하는 컴포넌트다. 
외부 트래픽이 쿠버네티스 서비스로 정상적으로 라우팅 되기 위해서는 위 두 컴포넌트가 완벽하게 설정되어 있어야 한다. 

NGINX는 고성능 웹 서버로, Ingress Controller 로 주로 사용된다. 그 이유는 NGINX의 다음과 같은 기능 때문이다.
- 웹 소켓 어플리케이션의 로드 밸런싱
- HTTPS 어플리케이션의 로드 밸런싱을 지원하는 SSL 기능
- 어플리케이션에 요청을 보내기 전에 URI을 재작성할 수 있도록 해주는 기능
- Session Persistence 기능(유료 버전에서만 제공)으로 한 클라이언트에서 들어오는 모든 요청을 하나의 백엔드 컨테이너로만 들어가게 해줌.
- JWTs (유료 버전에서만 제공), JSON Web Token 을 검증하여 요청을 인증하는 기능. 


![](https://gcpstaging-qwiklab-website-prod.s3.amazonaws.com/bundles/assets/1356d6cd1e7d299f188e5c338fd266e4cd77f0bd49c27ff2758f5c65d0f60385.png)

0. 기본 설정
- 존 설정
- k8s 클러스터 생성
- Helm 설치 & Init
- Tiller 계정 생성 및 deploy

1. NGINX-INGRESS 설치
`helm install --name nginx-ingress stable/nginx-ingress --set rbac.create=true`

2. 설치된 INGRESS service 확인
```
RESOURCES:
==> v1/Service
NAME                           TYPE          CLUSTER-IP     EXTERNAL-IP  PORT(S)                     AGE
nginx-ingress-controller       LoadBalancer  10.15.241.226  <pending>    80:30909/TCP,443:31829/TCP  1s
nginx-ingress-default-backend  ClusterIP     10.15.249.132  <none>       80/TCP                      1s
```
nginx-ingress-default-backend. The default backend is a Service which handles all URL paths and hosts the NGINX controller.

3. INGRESS resource 설정
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-resource
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /hello
        backend:
          serviceName: hello-app
          servicePort: 8080
```
* `kind: Ingress` 룰 설정하는 파일임을 알려줌.
* `spec` 부분에 path와 거기에 연결할 서비스 이름과 포트를 넣어줌.
* 설정을 마치고 나면 `kubectl apply -f ingress-resource.yaml`로 실제 적용해주는 거 잊지 말기

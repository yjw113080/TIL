## Orchestrating the Cloud with Kubernetes
1. Pod
![](https://gcpstaging-qwiklab-website-prod.s3.amazonaws.com/bundles/assets/b73bcce701677c046738d0175fdea7cfc3a0a8e6b58a1c7a90293e7a531a91fc.jpg)
Pod는 하나 이상의 컨테이너들의 집합이다. 상호 의존관계에 있는 컨테이너가 여럿 있을 경우, 이 컨테이너들을 하나의 pod로 패키징한다.
Pod는 필연적으로 해당 Pod가 위치한 Node (Host OS)에 의존한다. 즉, Node가 죽으면 Pod도 죽는다.
Pod는 디스크 볼륨을 갖는데, pod가 살아 있을 경우에만 유효하다. pod 안의 컨테이너들은 이 디스크에 접근/사용할 수 있다. Pod는 또한 공통된 네임스페이스를 제공하고, 컨테이너들은 이를 기반으로 서로 통신하고 볼륨을 공유할 수 있다.


Pod 는 yaml 컨피그 파일에 의해 생성된다.
1.1. `[pod이름].yaml` 예시
```
apiVersion: v1
kind: Pod
metadata:
  name: monolith
  labels:
    app: monolith
spec:
  containers:
    - name: monolith
      image: kelseyhightower/monolith:1.0.0
      args:
        - "-http=0.0.0.0:80"
        - "-health=0.0.0.0:81"
        - "-secret=secret"
      ports:
        - name: http
          containerPort: 80
        - name: health
          containerPort: 81
      resources:
        limits:
          cpu: 0.2
          memory: "10Mi"
```
1.2. `kubectl create -f [pod이름].yaml` 으로 pod 생성


1.3. Port-Forwarding 하기
`kubectl port-forward monolith OS에접근하는포트:컨테이너에접근하는포트`


2. Services
Pod는 서비스 장애 등등의 이유로 재시작될 수 있으며, 그 때마다 external IP가 초기화됨. 이런 상황에서도 pod 끼리, 외부로 통신할 수 있게 하기 위해 Service가 등장.
![](https://gcpstaging-qwiklab-website-prod.s3.amazonaws.com/bundles/assets/260d13ff7dba012c2a783d6f0143c1587e70d43ff4a199facf9990e4cb9bc0bf.jpg)

2.1. LABELS
Services는 자신과 연결된 pod를 식별하기 위해 label을 사용한다. Pod에 정확한 라벨이 붙어 있다면 Services가 자동으로 그 Pod 들을 식별해내서 통신을 오픈(`expose`)해준다.

Pod 생성 yaml 파일
```
kind: Service
apiVersion: v1
metadata:
  name: "monolith"
spec:
  selector:
    app: "monolith"
    secure: "enabled"
  ports:
    - protocol: "TCP"
      port: 443
      targetPort: 443
      nodePort: 31000
  type: NodePort
```
services에서 확인할 label이 `spec.selector` 부분이다.
호스트 OS의 31000 포트로 들어오는 요청을, 컨테이너의 443 포트로 받는다.
이때 외부 통신을 가능하게 하기 위해서는 클라우드 프로바이더 측의 방화벽 오픈이 필요함을 잊지 말아야 한다!!

2.2. LABLE 확인하기
```
kubectl label pods POD이름 'XXXX'
kubectl get pods secure-monolith --show-labels
```
위 커맨드를 통해 라벨링이 정확히 된 것을 확인했다면 `kubectl describe services monolith | grep Endpoints` 명령어를 실행했을 때 endpoint가 잡혀야 한다.


3. Deployments
![](https://gcpstaging-qwiklab-website-prod.s3.amazonaws.com/bundles/assets/7c7e19c4637183928b072e7291bc0a3484bd30827d72070c103ba10746bdb81a.jpg)
Deployments 는 애초에 계획된 만큼의 Pod가 정상적으로 작동하도록 하는 방식이다. 예를 들어 Node3가 죽어서 그 안에 있던 Pod3가 같이 죽었을 경우, K8S는 Node를 새로 찾고 하는 게 아니라 그냥 옆에 다른 노드로 옮겨가서 그 Pod를 또 만들도록 해준다.
Pod를 관리하기 위한 아주 상세한 기술들(HA...)을 알 필요 없다. Deployments 가 내장하고 있는 Replica Set을 이용하여, 해당 부분을 추상화하여 제공해주기 때문. 




## Managing Deployments with Kubernetes Engine
와 진짜 내가 궁금했던 부분이다. MultiCloud를 쓸 때 한 어플리케이션을 어떻게 서로 다른 리전에 효율적으로 자동화하여 배포하고 관리할 것인지.

1. Deployment 구조 확인
`kubectl explain deployment.metadata.name`

2. Scale Up / Down
`kubectl scale deployment hello --replicas=3`

3. 대망의 Rolling update 배포하기
아니 너무 쉬워서 깜짝 놀람;;

3.1. Deployment 설정 수정하기
`kubectl edit deployment XXXXXX
```
...
containers:
- name: XXXXXX
  image: [hostname]]/XXXXXX:2.0.0  // 이 부분에 업데이트 반영된 이미지를 넣는다.
...
```

3.2. 업데이트 확인
`kubectl get replicaset`
3.1.에서 설정 파일을 수정하면 자동으로 반영되어 K8S에서 배포함. 이 내용을 확인하기 위해 위 명령어 수행.
* `kubectl rollout history deployment/hello` 히스토리 확인
* `kubectl rollout pause deployment/hello` 업데이트 중 문제 발견했을 때 일시정지하기
* `kubectl rollout status deployment/hello` 현재 상태 확인하기
* `kubectl rollout resume deployment/hello` 업데이트 다시 시작하기
* `kubectl rollout undo deployment/hello` 롤백하기

4. Canary Deployment
![](https://gcpstaging-qwiklab-website-prod.s3.amazonaws.com/bundles/assets/a92ae020fe45c962846f03a4dcf30f0002c9b50a09a04a6024c5706ae65a668c.png)
프로덕션 환경에서 새로운 기능을 도입할 때, 일부 유저를 대상으로만 테스트하고 싶을 때 Canary Deployment를 한다. 새 릴리즈에서 발생할 수 있는 리스크를 최소화하기 위한 방법.
4.1. 설정 파일에 Track 넣어주기
Canary 배포 대상 이미지는 아래와 같이 `track: canary` 라고 추가해주어야 한다.
```
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: hello
        track: canary
        version: 2.0.0
```

4.2. SessionAffinity
Canary Deployment를 할 때 유의해야 하는 점은 들어오는 요청의 일부가 랜덤으로 선정된다는 점이다. 어떤 유저가 처음 접속할 때는 일반 배포분을, 그 다음에는 카나리 버전을, 그 다음에는 또 일반 배포분을 경험할 수도 있다는 이야기다. 유저들의 혼란을 없애기 위해서는 한 유저가 한 버전 만을 경험하도록 해야 하는데, 이 때 사용하는 것이 SessionAffinity 이다.
```
kind: Service
apiVersion: v1
metadata:
  name: "hello"
spec:
  sessionAffinity: ClientIP
  selector:
    app: "hello"
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 80
```

5. Blue-Green Deployment
롤링 업데이트가 오버헤드를 최소화하여 조금씩, 성능 저하 없이 어플리케이션을 배포할 수 있다는 점에서는 좋다. 그렇지만 모든 게 완벽히 배포된 뒤에만 로드 밸런서를 설정해줘야 하는 경우가 있다. 이럴 때 필요한 것이 Blue-Green 배포.
자원이 두 배가 든다는 문제가 있지만 안정성을 위해 고려해볼 만한 패턴.

5.1. 기존 deploy 분(hello-blue)을 프로덕션으로 적용한다.
`kubectl apply -f services/hello-blue.yaml`

5.2. Green 버전에 대한 yaml을 새로 생성한다.
`track: stable`에 유의한다!
```
metadata:
  name: hello-green
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: hello
        track: stable
        version: 2.0.0
```

`kubectl create -f deployments/xxxx.yaml`로 배포한다.
```curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version```배포분에 대한 테스트를 수행한다.


5.3. 새로 생성된 Green 버전을 프로덕션으로 받는다.
`kubectl apply -f services/hello-green.yaml`

5.4. Blue 버전으로 롤백도 가능하다.
`kubectl apply -f services/hello-blue.yaml`

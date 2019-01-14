Kubernetes in the Google Cloud 퀵랩을 시작했다.
Docker와 K8S에 대해서 얄팍하게 알고 있던 것들을 손으로 쳐보면서 조금 더 확실히 이해할 수 있게 된 것 같다. 

## Introduction to Docker
* How to push Docker images to Google Container Registry.
컨테이너 실행하고 빌드하는 건 해봤는데 export 하는 건 처음 해봐서 좋았다.
* 참고: Dockerfile 작성 예시
```
cat > Dockerfile <<EOF
# Use an official Node runtime as the parent image
FROM node:6

# Set the working directory in the container to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Make the container's port 80 available to the outside world
EXPOSE 80

# Run app.js using node when the container launches
CMD ["node", "app.js"]
EOF
```

* 남겨두고 싶은 CMD 몇 줄
`gcloud docker -- push gcr.io/[project-id]/node-app:0.2`



## Hello Node Kubernetes
![](https://gcpstaging-qwiklab-website-prod.s3.amazonaws.com/bundles/assets/641fcb4eefdf381baeeecbda634fe27abfd2bdb25d7c5dcbadbff9c285de79c2.png)

* Google Cloud 에서 컨테이너 클러스터 생성하기
```
gcloud container clusters create hello-world \
                --num-nodes 2 \
                --machine-type n1-standard-1 \
                --zone us-central1-a
```
Kubernetes Engine 클러스터를 먼저 생성해준다. 즉 실제 내 어플리케이션 컨테이너 클러스터를 생성할 쿠버네티스 환경을 만들어주는 것.
구글이 호스팅하는 K8S 마스터 API 서버의 클러스터로 이해하면 된다.
이때 VM을 생성한다고 생각해서 의아했는데, `n1-standard-1`은 컨테이너 클러스터 생성할 때 어떤 VM을 이용할 것인지 정하는 거였다.

* K8S Pod 생성하기
```
kubectl run hello-node \
    --image=gcr.io/PROJECT_ID/hello-node:v1 \
    --port=8080
                ```
실제 내 어플리케이션의 클러스터가 생성된다. 이 명령어를 실행하면 `hello-node`라는 이름의 Deployment 가 생성이 된다.
말 그대로 구현체 덩어리라고 생각하면 된다.

* K8S 외부 트래픽 열기 == service 생성
`kubectl expose deployment hello-node --type="LoadBalancer"`
이렇게 하면 등록된 Deployment 가 외부로 오픈된다. `kubectl get services`를 확인하면 
```
NAME         CLUSTER-IP     EXTERNAL-IP      PORT(S)    AGE
hello-node   10.3.250.149   104.154.90.147   8080/TCP   1m
kubernetes   10.3.240.1     <none>           443/TCP    5m
```
이런 식으로 External-IP가 생성된 것을 볼 수 있다.

궁금한 점, `expose` 할 때 외부 IP를 어떻게 뿅하고 바로 할당해주지? DHCP로 넣어주는건가 싶었는데 `kube-proxy`라는 내장 엔진이 IP 할당을 관장한다고 한다.
IP 범위도 한정지을 수 있고, 원래 갖고 있던 DNS랑 연결해서 사용할 수도 있기도 하다.


* K8S로 Scale up 하기
`kubectl scale deployment hello-node --replicas=4`
너무 쉬워서 깜짝 놀랐다.
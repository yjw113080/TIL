서비스가 시작된 뒤로 계속 실행되는 상태를 유지하기 위해서는, 도커 컨테이너를 모니터링 하고 멈추면 재시작해주는 또 다른 서비스가 있어야 함. k8s가 그 역할을 대신해줌. 

## Running a MongoDB Database in Kubernetes with StatefulSets

1. StorageClass 생성하기
`StorageClass` DB 노드들이 어떤 데이터 저장소를 사용하는지 지정.
`kubectl apply -f googlecloud_ssd.yaml` yaml 파일 설정을 이용해 `StorageClass` 생성.



2. 서비스 설정하기
2.1. Headless Service
* K8S에서 서비스란 특정 pod에 접근하는 정책/룰을 의미한다. Headless service는 load balancing 을 사전에 정의하지 않는 서비스.
* StatefulSets와 함께 사용되면 DNS 각각이 pod에 접근할 수 있게 되고, 모든 MongoDB 노드에 개별적으로 접근할 수 있다. 
* yaml 파일에 해당 서비스가 Headless 라는 설정을 넣기 위해서는 `clusterIP: None` 값을 주어야 한다.


2.2. StatefulSet
1) 개념
* StatefulSet 이란 stateful 한 어플리케이션을 관리하기 위한 k8s의 API다.
* Stateful app 이란 한 세션 내의 활동으로부터 클라이언트의 정보를 저장하여 다음 세션에서 이용하는 앱이다. 이때 저장된 데이터는 어플리케이션의 state라고 불린다.

2) yaml 파일 설정
* yaml 파일에 `kind: StatefulSet` 을 넣어 statfulset의 설정파일임을 선언한다.
* DB의 안정적인 shutdown을 위해 `terminationGracePeriodSeconds: xx` 값을 준다.
* Mongo DB가 데이터를 적재할 경로에 저장소를 마운트해준다.
```
          volumeMounts:
            - name: mongo-persistent-storage
              mountPath: /data/db
```
* 1에서 생성한 `StorageClass`과의 인터페이스를 정의해준다. MongoDB replica 각각에 대해 100GB의 디스크를 할당하도록 한 것.
```
  volumeClaimTemplates:
  - metadata:
      name: mongo-persistent-storage
      annotations:
        volume.beta.kubernetes.io/storage-class: "fast"
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 100Gi
```


2.3. 서비스 시작하기
`kubectl apply -f mongo-statefulset.yaml`
항상 `apply -f` 해서 실제 서비스 생성하는 거 잊지 않기!



3. 실제 내 어카운트로 실습할 때 clean-up 하는 방법
`kubectl delete statefulset mongo` stateful set을 지우고
`kubectl delete svc mongo` 서비스를 지우고
`kubectl delete pvc -l role=mongo`
`gcloud container clusters delete "hello-world"` GCP의 클러스터를 지운다.




## Build a Slack Bot with Node.js on Kubernetes

1. `Secret`
secret은 k8s 에서 제공하는 API로, 패스워드, 토큰, 키 등의 민감 정보를 담는다.

yaml 파일에 문제가 있어서 수정하고 나서는, pod를 어떻게 리프레시 시키지? `apply` 다!!!!

2. CrashLoopBackOff
클러스터 실행할 때 `kubectl get pods` 해봤더니 이 에러가 계속 났다.
찾아보니 pod를 실행하려고 할 때마다 계속 crash가 나서 무한으로 계속 실행시키려다 실패하고를 반복하고 있는 상태라고 한다.
`docker logs [컨테이너ID]` docker 컨테이너 자체의 크래시 이유를 찾아야 했다.
들여다보니 node 파일에서 blocked scope을 선언하는 `let`이 지원이 안된다는 거였다.
`docker run` 뒤에 크래시 여부를 잘 봐야 했는데 진도 나가기에 급해서 제대로 살피지 않은 것이 화근이었다.

* Tip today
커맨드 출력값을 변수에 담아서 쓰기 `export PROJECT_ID=$(gcloud config list --format 'value(core.project)')`
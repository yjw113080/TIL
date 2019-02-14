
## Application Load Balancer
![](https://us-west-2-tcprod.s3.amazonaws.com/courses/ILT-TF-100-ARCHIT/v6.1.0/lab-3-ha/instructions/en_us/images/load-balancing.png)

여러 서버에서 앱이 실행되기 때문에 그 상단에서 로드 밸런싱을 해줘야 한다.
AWS에는 세 가지의 로드 밸런서가 있는데, 1) Application Load Balancer 2) Network Load Balancer 3) Classic Load Balancer 다. 1)은 응용 계층(OSI)에서, 2)는 네트워크 계층에서 로드 밸런싱한다. 다른 것들도 많지만, ALB는 자체적으로 누가 들어올 수 있고 들어 올 수 없는지 정할 수 있다는 점이 2) 3) 대비 장점이다. 


## Auto Scaling 그룹 생성
사용자가 정의한 정책, 일정, 상태 확인에 따라 자동으로 EC2 인스턴스를 시작/종료한다.
여러 AZ에 인스턴스를 자동으로 분산하여 고가용성 앱을 생성한다.
private subnet에 앱을 배포하여, 외부에서는 인스턴스에 직접 접근할 수 없도록 하되 로드 밸런서에 요청을 보내면 받아주도록 설정한다.
`Keep this group at its initial size` 설정을 통해 항상 같은 수의 인스턴스 이미지를 유지하도록 한다. 여기서 설정을 바꾸면 오토 스켈링이 가능하다.


## 보안 그룹 업데이트
![](https://us-west-2-tcprod.s3.amazonaws.com/courses/ILT-TF-100-ARCHIT/v6.1.0/lab-3-ha/instructions/en_us/images/security.png)

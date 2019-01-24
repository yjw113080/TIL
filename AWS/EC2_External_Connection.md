## EC2에 mysql 깔고 외부 연결하기
side-project 팀과 함께 DB를 놓고 개발하고 싶어서 AWS Free-Tier 계정으로 서버를 하나 만들었다.
기본적으로 ubuntu 18.04에 mysql-server 설치를 했고 외부 통신을 가능하게 하기 위해 다음과 같은 작업을 해줘야 했다.

1. AWS 클라우드 차원의 방화벽 오픈
자신이 만든 EC2 이미지가 어떤 보안 그룹에 속하는지 확인한 뒤 `Network & Security` 메뉴의 `보안 그룹`에 들어간다.
`인바운드 규칙`에서 `3306` 포트를 `위치 무관`으로 오픈하는 룰을 추가한다.

2. 이제 서버의 mysql 설정을 넣어줘야 할 차례
`/etc/mysql/mysql.conf.d/mysqld.cnf` 를 열어서
`bind-address = 0.0.0.0` 을 추가해준다.
`service mysql restart` 수행하여 변경된 설정값을 적용해준다.


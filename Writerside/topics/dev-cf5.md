# 개발참고5

### 젠킨스 설치 ###

jenkins.war 업로드
java -jar jenkins.war --httpPort=7070

source /usr/lib/systemd/system/jenkins.service

sudo systemctl daemon-reload

sudo systemctl restart jenkins

//서버 시작시 스타트
sudo systemctl enable jenkins

sudo chown -R lgbadmin:xadm jenkins

sudo chown -R lgbadmin: /var/lib/jenkins
sudo chown -R lgbadmin: /var/cache/jenkins
sudo chown -R lgbadmin: /var/log/jenkins

sudo vi etc/profile
> export JAVA_HOME=$(readlink -f /usr/bin/java | sed "s:bin/java::")
source etc/profile

echo $JAVA_HOME
echo $PATH

- yum pacakge 삭제
  sudo yum -y remove java-17-amazon-corretto.x86_64

- 패키지 삭제
  sudo rpm -e jenkins-2.452.2-1.1.noarch

- 설치된 전체 패키지 정보 확인
  rpm -qa
  rpm -qa | grep jenkins

- default java 변경
  alternatives --config
  sudo alternatives --install /usr/bin/java java /engn001/jdk-17.0.2/bin/java 1
  readlink -f /usr/bin/java
  alternative --list
  alternatives --list
  alternatives --disaply java
  alternatives --display jre_17
  alternatives --display java
  alternatives --remove jre_17
  readlink -f /usr/lib/jvm/java-17-amazon-corretto.x86_64
  alternatives --remove jre_17 /usr/lib/jvm/java-17-amazon-corretto.x86_64
  sudo alternatives --remove jre_17 /usr/lib/jvm/java-17-amazon-corretto.x86_64
  alternatives --list


### Jenkins 세팅 ###
1. Pipe Line 선택

2. Advanced Project Option
    - Defintion
      : PipeLine script from SCM
      SCM : Git
      RepositoryUrl : http://10.94.57.6:8080/project/enoi-2024/enoi-ex-fe.git
      Credentials : enoi_jenkins/******	>> 미리 젠킨스용 계정 생성
      ScriptPath : cicd/Jenkinsfile_dev

-- Jenkins 버전 업데이트
cf : https://jojoldu.tistory.com/514
ps -ef | grep jenkins 를 통해 War파일 위치 확인
> 해당 war 파일을 신규 War 파일로 교체 후 재시작 : sudo systemctl restart jenkins


### Jenkins Tag 세팅 ###
Configure - General
이 빌드는 매개변수가 있습니다 체크
Name : GIT_TAG
ParameterType : Tag
고급 >
Barnch : 공백
Branch Filter : .*
Tag Filter : refs/tags/tags*
Sort Mode : DESCENDING_SMART
Selected Value : TOP


### 젠킨스 업데이트 에러로그 방지 ###
systemctl status jenkins 통해 jenkins.service 위치 확인

sudo vi /usr/lib/systemd/system/jenkins.service

Environment="JAVA_OPTS=-Djava.awt.headless=true -Dhudson.model.UpdateCenter.never=true"
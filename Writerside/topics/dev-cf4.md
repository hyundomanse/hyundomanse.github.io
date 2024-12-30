# 개발참고4

### 계정정보

LENA : admin / !admin1234
SFTP, SSH : lgbadmin / !dpfwldps0

### 리눅스 명령어 ###

- Linux 정보 확인
  cat /proc/version

- OS 정보 확인
  cat /etc/issue*
  cat os-release

- OS 상세 확인
  cat /etc/*release*

- 비트 확인
  getconf LONG_BIT

- 리소스 사용량 확인
  top
  s 이후 0.5 입력 : 0.5초마다 갱신
  df -h

- 루트 경로 이동
  cd /

- 사용자 루트 경로 이동
  cd ~/

- 권한 부여
  sudo chmod 777 (or 665) -R ../engn001/
  -R 옵션은 하위까지 부여할때에 사용

- java 설치 경로 확인
  which java

- 로그인 계정 확인
  whoami

- 소유권한 부여
  sudo chown -R lgbadmin:xadm engn001
  sudo chown -R lgbadmin:xadm /var/cache/jenkins
  sudo chown -R lgbadmin:xadm /var/lib/jenkins

- 동작 확인
  ps -ef | grep jenkins

rpm -qa | grep jenkins

- 심볼릭 링크 연결되어있는 파일의 경로 확인
  readlink -f /usr/bin/java
  readlink -f는 심볼릭 링크에서 원본파일을 추출하는 명령어

- 명령어 이력 확인
  history
  history | grep -A5 -B10 build		
  history | grep -A5 -B10 rm
  :: -A num, --after-context=num, -B num, --before-context=num
  :: Print num lines of leading context before each match.

- 압축 풀기
  tar xvf jdk-8u202-libux-x64.tar.gz

- 복사 	
  cp -rp test.txt /engn001/

- 이동
  mv test.txt lenaw

- 이름 변경
  mv web2 web8080

- 폴더 삭제
  rmdir web2
  rm -rf web2

- 방화벽 확인
  nc -vz 10.94.56.58 16100
  결과값 >>
  Ncat: Version 7.93 ( https://nmap.org/ncat )
  Ncat: Connected to 10.94.56.58:16100.
  Ncat: 0 bytes sent, 0 bytes received in 0.04 seconds.

- 포트 체크 방법
1.
yum install -y nc
nc -z 주소 포트 > TCP 체크
nc -zu 주소 포트 > UDP 체크

2.
yum install -y nmap
sudo nmap -PN 10.94.56.58 -p 16100 > TCP 확인
sudo nmap -sU 10.94.56.58 -p 16100 > UDP 확인

- SSL 버전 확인
  openssl version

### 레나 설치 ###

- 레나 설치중 lib 이슈 있는 경우 확인
  sudo yum install -y apr-util-devel
  sudo yum install -y libxcrypt-compat
  ldd rotatelogs

- 레나 서비스 등록
  /engn001/lena/1.3/bin

sudo ./service-manager.sh register
sudo ./service-agent.sh register
sudo ./service.sh register

>> cf

cd /engn001/lena/1.3/bin
sudo ./service-manager.sh register
sudo ./service-agent.sh register

cd /engn001/lena/1.3_en10/bin
sudo ./service-agent.sh register

cd /engn001/lena/1.3_en10/servers/enoi_9040/
sudo ./service.sh register

cd /engn001/lenaw/1.3/servers/enoi_80/
sudo ./service.sh register


- yarn 설치 및 넥서스 세팅
  sudo npm -g config set registry http://10.94.57.6:8081/repository/npm-group/

sudo npm -g install yarn
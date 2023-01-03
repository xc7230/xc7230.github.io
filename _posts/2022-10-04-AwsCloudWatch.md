---
title: '[AWS] CloudWatch'
categories:
    - AWS
    - Cloud
    - Monitoring
tag:
    - AWSCloud
    - Cloud
    - Monitoring


last_modified_at: 2022-10-04T09:00:00+09:00
toc: true
---
AWS 클라우드 서버 모니터링<br/>

# CloudWatch
![image](/assets/img/image/cloudwatch/1.png)<br/>

## CloudWatch 구성하기
- EC2 생성

### 모니터링 세팅
![image](/assets/img/image/cloudwatch/2.png)<br/>
![image](/assets/img/image/cloudwatch/3.png)<br/>
세부모니터링을 활성화 한다.<br/>

### 지표 보기
![image](/assets/img/image/cloudwatch/4.png)<br/>

### 경보 생성하기

![image](/assets/img/image/cloudwatch/6.png)<br/>
![image](/assets/img/image/cloudwatch/7.png)<br/>
![image](/assets/img/image/cloudwatch/9.png)<br/>
![image](/assets/img/image/cloudwatch/10.png)<br/>
이메일 인증을 해야한다.<br/>

### SNS 등록하기
![image](/assets/img/image/cloudwatch/8.png)<br/>


### 확인
생성한 EC2에 부하를 줘본다.<br/>
![image](/assets/img/image/cloudwatch/11.png)<br/>
![image](/assets/img/image/cloudwatch/12.png)<br/>
![image](/assets/img/image/cloudwatch/13.png)<br/>
다음과 같이 설정한 cpu 사용량을 넘으면 e-mail로 메일이 온다.<br/>


###  대시보드
![image](/assets/img/image/cloudwatch/14.png)<br/>
![image](/assets/img/image/cloudwatch/15.png)<br/>
위젯을 추가하여 대시보드를 꾸밀 수 있다.<br/>

### EC2정보 CloudWatch로 보내기

- EC2 패키지 다운
```shell
apt update
apt install -y awscli
aws --version #확인
```

- 계정정보 설정
```shell
aws configure list
```
![image](/assets/img/image/cloudwatch/17.png)<br/>
![image](/assets/img/image/cloudwatch/18.png)<br/>
엑세스키 생성을 해야 한다.<br/>
```shell
aws configure
```
엑세스키 정보와 지역(` ap-northeast-2 `)을 입력한다.

![image](/assets/img/image/cloudwatch/19.png)<br/>

- EC2로 지표 생성하기
```shell
aws cloudwatch put-metric-data --metric-name "World" --namespace "Hello" --value 2 #누를때 마다 생성되니 한 번만 누르자
```
![image](/assets/img/image/cloudwatch/20.png)<br/>

## Shell 프로그래밍
리눅스에서 사용하는 명령어를 가지고 프로그래밍한다. 반복적인 작업을 자동화 한다.<br/>
### 변수
```shell
vi ex01.sh
```

- 쉡 스크립트 생성
```Shell Script
#!/bin/bash #명령어를 어떤 쉘로 실행시킬지 지정

ls  #실행시키고 싶은 명령어
```

- 쉘 스크립트 실행하기
```shell
chmod 755 ex01.sh   #권한 설정
/root/ex01.sh   #쉘 스크립트 실행
```

- 쉘 스크립트 변수만들기
```shell
vi ex02.sh
```

```Shell Script
#!/bin/bash

VAR1=10

echo $VAR1
```
- 쉘 스크립트 실행하기
```shell
chmod 755 ex02.sh   #권한 설정
/root/ex02.sh   #쉘 스크립트 실행
```

```Shell Script
#!/bin/bash

VAR1=10
VAR2=20

RESULT=`expr $VAR1 + $VAR2`

echo $RESULT
```
- 행, 열 뽑기
```shell
free    #free의 Mem 행에 total열을 뽑아본다.
31-46-168:~# free
              total        used        free      shared  buff/cache   available
Mem:         997500      146484      160168         780      690848      698124
Swap:             0           0           0

free | grep Mem | awk -F " " '{print $2}'
```


- 쉘 스크립트로 만들어 보기
```shell
vi ex04.sh
```
```Shell Script
#!/bin/bash

TOTAL=`free | grep Mem | awk -F " " '{print $2}'`

echo $TOTAL
```
```shell
chmod 755 ex04.sh   #권한 설정
/root/ex04.sh   #쉘 스크립트 실행
```

```Shell Script
#!/bin/bash

TOTAL=`free | grep Mem | awk -F " " '{print $2}'`
USED=`free | grep Mem | awk -F " " '{print $3}'`

RESULT=`expr $USED \* 100`
RESULT=`expr $RESULT \/ $TOTAL`

echo $RESULT
```

### 메모리 사용 EC2에 보내기

```Shell Script 
#!/bin/bash

TOTAL=`free | grep Mem | awk -F " " '{print $2}'`
USED=`free | grep Mem | awk -F " " '{print $3}'`

RESULT=`expr $USED \* 100`
RESULT=`expr $RESULT \/ $TOTAL`

echo $RESUL
aws cloudwatch put-metric-data --metric-name "MemoryUsage" --namespace "EC2" --value $RESULT
```
```shell
/root/ex04.sh
```
![image](/assets/img/image/cloudwatch/21.png)<br/>


### 반복작업 스케쥴링
```shell
crontab -e  # 명령어를 실행하면 편집기가 실행되고 편집기를 이용해 내용을 작성한다.
```

```shell
crontab -e
```
```shell
* * * * *       /root/ex04.sh #1분마다 ex04.sh를 실행한다.
```
```shell
crontab -l
```
![image](/assets/img/image/cloudwatch/22.png)<br/>


### 온프레미스로 설정해보기
- 프로메테우스 : 메트릭 수집 프로그램
- 그라파나 : 시각화 도구
- 알람 : 메일 서버, DNS 서버

#### 메일 서버 만들기
```shell
apt install -y postfix # no configration 선택
vi /etc/hostname
```
```shell
mail.cloudcampkjh.kro.kr #내가 만든 도메인 주소 넣기
```
```shell
cp /etc/postfix/main.cf.proto /etc/postfix/main.cf
vi /etc/postfix/main.cf
```
https://www.server-world.info/en/note?os=Ubuntu_18.04&p=mail&f=1 을 참조하여 파일 수정<br/>
- 187번 라인 참조
![image](/assets/img/image/cloudwatch/23.png)<br/>
111.111.0.0/24 형식으로 저장<br/>

```shell
newaliases
systemctl restart postfix
netstat -anlp | grep :25
```
![image](/assets/img/image/cloudwatch/24.png)<br/>

```shell
apt -y install dovecot-core dovecot-pop3d dovecot-imapd
```
https://www.server-world.info/en/note?os=Ubuntu_18.04&p=mail&f=2 참조해서 내용 수정<br/>

```shell
systemctl restart dovecot
netstat -anlp | grep :143
```
![image](/assets/img/image/cloudwatch/25.png)<br/>

- 확인<br/>
```shell
apt -y install mailutils
echo 'export MAIL=$HOME/Maildir/' >> /etc/profile.d/mail.sh
useradd -s /bin/bash -m test01
passwd test01
useradd -s /bin/bash -m test02
passwd test02
su - test01
```
- 메일 보내기<br/>
```shell
mail test02@localhost
		Cc:  #엔터
		Subject: Test Mail   #입력하고 엔터
		This is the first mail.    #입력하고 엔터
		#Ctrl + d로 작성 종료
exit
```
- 메일 확인<br/>
```shell
su - test02
mail
```
![image](/assets/img/image/cloudwatch/26.png)<br/>

#### 도메인 설정
한국에 할당받은 도메인 설정<br/>

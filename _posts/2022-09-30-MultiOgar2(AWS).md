---
title: '[AWS] MultiOgar2'
categories:
    - AWS
    - Cloud
tag:
    - AWSCloud
    - Cloud

last_modified_at: 2022-09-30T09:00:00+09:00
toc: true
---
# MultiOgarII
- https://github.com/m-byte918/MultiOgarII/tree/master/src 에 올라와 있는 세포 삼키기 게임을 AWS EC2를 이용해서 구현해라

## WAS설정
```shell
apt update

# 구현하는데 필요한 노드 JS 설치
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
apt-get install -y nodejs

# git에 있는 코드 다운
git clone https://github.com/m-byte918/MultiOgarII.git  #git코드 그대로 받아서 쓰기


# 다운 받은 폴더로 이동
cd MultiOgarII

# 안에 있는 노드JS 코드 인스톨
npm i

# 게임 서버 실행
cd src
node index.js # 포트번호 8080으로 성공이 된다.
```
![image](/assets/img/image/game/5.png)<br/>

## 포트번호 설정
![image](/assets/img/image/game/2.png)<br/>
게임 서버의 포트번호(8080)을 추가해준다.

## WEB 설정

- 세포 삼키기의 게임 서버를 열렸다. 이제 그것을 웹으로 출력해줄 웹서버를 만들어야 한다.

- https://github.com/Luka967/Cigar 이라는 git에서 세포키우기의 웹서버를 만들어 줬다. 이걸 활용한다.

```shell
apt update
apt install -y apache2
systemctl restart apache2 #서버 작동하는지 확인

# git에서 코드 가져오기
git clone https://github.com/Luka967/Cigar.git


# 로그인 파일 이동
cd Cigar/www/
mv * /var/www/html/

# 파일 수정

vi /var/www/html/index.html
```
- index.html<br/>
![image](/assets/img/image/game/1.png)<br/>
다음과 같이 인덱스 57번째 줄에 Was의 주소와 포트 번호(8080)를 추가해준다.<br/>

## 결과
```shell
# 웹서버를 한 번 재시작 해준다.
systemctl restart apache2
```
WEB서버의 주소로 접속해 보면 다음 화면이 출력된다.<br/>

![image](/assets/img/image/game/3.png)<br/>

그리고 옆에 서버목록에 내가 만든 서버가 올라와 있다.<br/> 
내 서버를 선택하고 Play를 눌러보면<br/>

![image](/assets/img/image/game/4.png)<br/>
다음과 같이 내 웹서버에 세포게임이 연결됐다.



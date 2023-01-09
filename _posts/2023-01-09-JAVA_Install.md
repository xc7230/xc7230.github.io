---
title: '[JAVA] JAVA 설치하기'
categories:
    - JAVA
tag:
    - JAVA
last_modified_at: 2023-01-09T00:01:00+09:00
toc: true
---

# JAVA
## 설치하기
### 다운로드
링크 : https://www.oracle.com/java/technologies/downloads/<br/>
zip 파일로 다운받아 C드라이브에 압축을 풀어준다.<br/>
![image](/assets/img/image/java_install/1.png)<br/>
![image](/assets/img/image/java_install/2.png)<br/>

### 환경변수 설정
JAVA 환경변수를 설정하여 내 윈도우 환경 어디서든 자바를 인식할 수 있도록 해준다.<br/>
![image](/assets/img/image/java_install/3.png)<br/>
![image](/assets/img/image/java_install/4.png)<br/>
다운받은 자바 폴더를 경로로 하는 `JAVA_HOME`이라는 변수 하나를 생성해준다.<br/>
![image](/assets/img/image/java_install/5.png)<br/>
`CLASSPATH`라는 변수명에, `%JAVA_HOME%\lib`경로를 입력해 변수를 추가해준다.<br/>
![image](/assets/img/image/java_install/8.png)<br/>
`Path`시스템 변수에 만든 `%JAVA_HOME%\bin`을 추가해준다.
![image](/assets/img/image/java_install/6.png)<br/>

### 확인
`CMD` 창에 `java -version`을 입력해 내가 다운 받은 버전이 나오면 성공이다.<br/>
![image](/assets/img/image/java_install/7.png)<br/>
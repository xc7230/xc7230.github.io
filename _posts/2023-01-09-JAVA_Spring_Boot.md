---
title: '[WEB] Spring Boot설치하기'
categories:
    - JAVA
    - Spring Boot
    - WEB
tag:
    - JAVA
    - Spring Boot
    - WEB
last_modified_at: 2023-01-09T00:01:01+09:00
toc: true
---


# JAVA SPRING BOOT
## 설치하기
### VS Code를 이용해 Spring Boot 설치
![image](/assets/img/image/java_spring_boot/1.png)<br/>
![image](/assets/img/image/java_spring_boot/2.png)<br/>

## Spring Boot 프로젝트 생성
단축키 `[Ctrl] + [Shift] + [P]` 입력해서 `[Command Palette]`을 실행한 후 `Spring Initializr`을 입력하여 Maven/Gradle 중 선택해주면(작성자는 Maven을 선택) 다음과 같은 과정이 나온다.
![image](/assets/img/image/java_spring_boot/3.png)<br/>
- 스프링 부트 버전<br/>
  ![image](/assets/img/image/java_spring_boot/4.png)<br/>
- 프로젝트 언어<br/>
  ![image](/assets/img/image/java_spring_boot/5.png)<br/>
- Group ID<br/>
  ![image](/assets/img/image/java_spring_boot/6.png)<br/>
- Artifact ID<br/>
  ![image](/assets/img/image/java_spring_boot/7.png)<br/>
- 패키지 타입<br/>
  ![image](/assets/img/image/java_spring_boot/8.png)<br/>
- 자바 버전<br/>
  ![image](/assets/img/image/java_spring_boot/9.png)<br/>
- 추가 라이브러리 설치
  ![image](/assets/img/image/java_spring_boot/10.png)<br/>
  이 작업은 나중에 해도 상관은 없다. 하지만 미리 설치해두면 생성시 자동으로 설정해준다.<br/>

## Spring Boot 프로젝트 실행해보기
### Index 파일 생성
![image](/assets/img/image/java_spring_boot/11.png)<br/>
`static` 폴더 안에 `index.html`파일을 새로 생성해준다.(`!`(느낌표)를 입력하면 html 기본 소스를 자동으로 입력해준다.)

### 실행
`demo/src/main/java.com.example.demo/SpringBootApplication.java` 클릭 후 `Run Java` 클릭으로 실행해준다.<br/>
![image](/assets/img/image/java_spring_boot/12.png)<br/>
![image](/assets/img/image/java_spring_boot/13.png)<br/>

### 확인
내 주소:8080(http://localhost:8080/)에 접속해보면 `index.html` 파일이 실행되는걸 확인 할 수 있다.<br/>
![image](/assets/img/image/java_spring_boot/14.png)<br/>

## 빌드
### 빌드 관리 도구
#### Maven
- 자바용 프로젝트 관리도구다.<br/>
- 메이븐은 내가 사용할 라이브러리 뿐만 아니라 해당 라이브러리가 작동하는데 필요한 다른 라이브러리까지 관리하여 네트워크를 통해 자동으로 다운해준다.<br/>
- XML 기반의 빌드처리를 작성한다. 복잡한 내용을 작성하게 되면 빌드처리가 어려워 진다.
  
#### Gradle
- 안드로이드 앱을 만들때 필요한 공식 빌드 시스템으로 JAVA 뿐만 아니라 C/C++, Python등을 지원한다.
- 메이븐의 XML 기반의 정적 스크립트가 아니고 동적 스크립트 형식이다.

### 플러그인 추가하기
#### DevTools
클래스 파일을 수정하고 테스트 하려면 실행되고 있는 서버를 수동으로 재시작해야 하는 번거로움이 있다. DevTools 같은 경우 자동으로 리로드 시켜주는 기능이 있다.<br/>
##### Gradle
`build.gradle`에 추가하고 `Refresh Gradle Project`
```gradle
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	developmentOnly 'org.springframework.boot:spring-boot-devtools'  //여기 추가
}
```
![image](/assets/img/image/java_spring_boot/15.png)<br/>

##### Maven
`pom.xml`에 추가하고 `Update Project`
```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-devtools</artifactId>
		<scope>runtime</scope>
		<optional>true</optional>
	</dependency>
</dependencies>
```
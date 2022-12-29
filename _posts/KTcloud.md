# KTcloud
## Server
### Server 생성하기
![image](./image/ktcloud/1.png)<br/>
![image](./image/ktcloud/2.png)<br/>
서버 이름, 사양, 요금제를 선택한 다음 생성을 한다.<br/>

### Network 생성하기
만약 KTcloud가 처음이라 server을 최초로 생성을 했으면 자동으로 네트워크가 하나 만들어진다. 추가로 만들고 싶으면 `IP생성`을 눌러준다.<br/>
![image](./image/ktcloud/3.png)<br/>
![image](./image/ktcloud/4.png)<br/>

### SSH Key Pair
![image](./image/ktcloud/5.png)<br/>
![image](./image/ktcloud/6.png)<br/>
위치를 server와 잘 맞춰 생성해주고 .PEM 파일을 잘 보관해둔다.<br/>
![image](./image/ktcloud/7.png)<br/>
![image](./image/ktcloud/8.png)<br/>
![image](./image/ktcloud/9.png)<br/>
`Putty`를 다운받으면 나오는 `PUTTYGEN`파일로 key파일을 PEM에서 PPK로 변경한다.<br/>

### Network Server에 연결하기
네트워크에 사설IP를 가진 서버와 연결하여 외부와 통신을 할 수 있게 만들어 준다. 방법은 두 가지로, `접속설정`을 이용해 포트로 통신하는 방법이고, 다른 하나는 `Static NAT`를 이용해 연결하는 방식이다.
- 접속설정
    ![image](./image/ktcloud/10.png)<br/>
    내가 만들어둔 server를 선택하고 포트를 남겨주면 된다. 22은 SSH의 포트번호로 Putty를 이용해 원격으로 서버를 조종 할 수 있게 만들어준다.

- Static NAT
    ![image](./image/ktcloud/11.png)<br/>
    ![image](./image/ktcloud/12.png)<br/>
    ![image](./image/ktcloud/13.png)<br/>
    ![image](./image/ktcloud/14.png)<br/>
    연결이 끝나면 방화벽에서 포트를 열어준다. 퍼블릭 환경이기 때문에 모든 ip를 열어둔다.<br/>

### Server에 원격 접속하기
![image](./image/ktcloud/15.png)<br/>
![image](./image/ktcloud/16.png)<br/>
server를 정지 후 상세 정보에서 SSH Key를 추가해준다음 실행해준다.<br/>
![image](./image/ktcloud/17.png)<br/>
![image](./image/ktcloud/18.png)<br/>
![image](./image/ktcloud/19.png)<br/>
![image](./image/ktcloud/20.png)<br/>
- 네트워크로 가서 연결된 네트워크의 ip를 저장한다.
- putty에 `root@네트워크ip`를 입력해준다.
- ppk파일을 선택한 다음 `open`을 누르면 KT cloud에서 생성한 server의 콘솔에 원격으로 접속할 수 있다.

## Load Balancer
만약 Load Balancer 서비스가 보이지 않는다면 `All Services`에 들어가 `서비스 신청`으로 `Load Balancer`를 활성화 해준다.<br/>
![image](./image/ktcloud/22.png)<br/>
![image](./image/ktcloud/21.png)<br/>

## WAF(웹방화벽)
### WAF 생성하기
![image](./image/ktcloud/23.png)<br/>
모두 기본 설정으로 생성<br/>

### WAF 설정하기
`Server-Networking`에 생성된 네트워크 중 기본형 네트워크에 포트포워딩 설정을 해준다.<br/>
![image](./image/ktcloud/24.png)<br/>
내가 생성한 WAF Name을 선택, 포트를 설정해준다.<br/>
![image](./image/ktcloud/25.png)<br/>
`Load Balancer`에 웹 서버가 아닌, 기본 네트워크에 연결한 WAF의 포트를 추가해준다.
![image](./image/ktcloud/26.png)<br/>

### WAF에 WEB Server 연결하기
WAPPLES 대시보드에 접속<br/>
![image](./image/ktcloud/27.png)<br/>
`프록시 IP`에서 WAF의 주소를 추가해준다.(IP는 WAF 상세정보 보기를 보면 있다.)
![image](./image/ktcloud/28.png)<br/>
`보호 대상 서버`에 다음과 같은 정보를 입력해주고 추가를 해준다. 웹 서버 IP에는 지난번 만들었던 Web server의 사설 IP를 넣었다.<br/>
![image](./image/ktcloud/29.png)<br/>
로드 밸런서 IP에 접속하면 WAF에 연결된 WebServer가 연결된다.<br/>
![image](./image/ktcloud/30.png)<br/>
도메인 까지 보안을 설정하고 싶으면 `정책 설정` 에서 도메인을 추가해준다.<br/>
![image](./image/ktcloud/31.png)<br/>





## Object Storage
### Storage 2.0 생성
만약 Storage 2.0 서비스가 보이지 않는다면 `All Services`에 들어가 `서비스 신청`으로 `Storage 2.0`를 활성화 해준다.<br/>

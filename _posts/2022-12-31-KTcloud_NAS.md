---
title: '[KTcloud] KTcloud를 이용한 Server와 NAS 연동'
categories:
    - Cloud
    - KTcloud
tag:
    - Cloud
    - KTcloud
    - NAS

last_modified_at: 2022-12-31T15:00:00+09:00
toc: true
---
KTcloud를 이용한 NAS<br/>

# NAS
## NAS 생성하기
![1](https://user-images.githubusercontent.com/33945185/210190870-cce3f341-9932-4b61-af11-1e396bdd5c9f.png)
<br/>
![2](https://user-images.githubusercontent.com/33945185/210190871-b39f1c99-4ae3-4230-8a76-834545cc85ce.png)
<br/>
![3](https://user-images.githubusercontent.com/33945185/210190872-2b1e9a74-f574-4264-9d06-5e3e506e7b24.png)
<br/>

## NAS Server 만들기
```shell
# 필요한 패키지 다운
yum install -y showmount
yum install -y nfs-utils

# nas server에 연동할 디렉토리 생성
mkdir /mynas

# nas mount 하기
mount -t nfs 172.27.128.1:mynas01 /mynas    # nas의 ip:mount path

# nas server 등록하기
vi /etc/fstab
# 추가
172.27.128.1:mynas01   /mynas       nfs     defaults        0 0
```
![5](https://user-images.githubusercontent.com/33945185/210190875-c2d07748-c3e1-476b-a11f-9ffe454d1b5f.png)
<br/>
### 연결 확인
```
df -h
```
![4](https://user-images.githubusercontent.com/33945185/210190874-217fbfe7-64e7-4fbb-be45-4d5b71bedd4a.png)
<br/>

## NAS Server에 연결하기
client server 준비<br/>
```shell
yum install -y showmount
yum install -y nfs-utils
mkdir /mynas
mount -t nfs 172.27.128.1:mynas01 /mynas
vi /etc/fstab
mount -t nfs 172.27.128.1:mynas01 /mynas
df -h # 연결 확인
```
## 2대의 server가 NAS server에 연결됐는지 확인
```shell
echo "Hello I'm nas" >> /mynas/nas.txt
cat /mynas/nas.txt
```
![6](https://user-images.githubusercontent.com/33945185/210190877-a0a6ee47-9b33-49f3-853f-4247bf594500.png)
<br/>

- 다른 client server에서 확인
    ![7](https://user-images.githubusercontent.com/33945185/210190878-a7e42358-44a4-4b9a-93ed-ba6ce3727b2c.png)
<br/>







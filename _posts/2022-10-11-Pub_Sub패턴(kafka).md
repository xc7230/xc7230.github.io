---
title: '[Linux] Kafka'
categories:
    - Linux
    - DATA
tag:
    - Linux
    - DATA


last_modified_at: 2022-10-11T09:00:00+09:00
toc: true
---
카프카를 이용한 데이터 분산<br/>
# Kafka

## 용어
- Topic
- Partiton
- Producer
- Consumer
- Broker
- Zookeeper
- Consumer Group
- Replication

## 설치
### 가상 OS 3개 설치
- broker : 192.168.197.10
- producer : 192.168.197.20
- consumer : 92.168.197.30

- 방화벽 해제(모두다)
```shell
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
```

- jdk설치(모두다)
```shell
yum -y install java-1.8.0-openjdk-devel.x86_64
```

- Kafka 설치(모두다)
```shell
yum install -y wget 
wget https://archive.apache.org/dist/kafka/3.1.0/kafka_2.13-3.1.0.tgz
tar -xzvf kafka_2.13-3.1.0.tgz
mv kafka_2.13-3.1.0 /opt/kafka
```
- 호스트 이름설정
    - 각 컴퓨터
```shell
vi /etc/hostname #각자 이름으로 바꾸고 재부팅
```

- 호스트 끼리 연결하기
```shell
vi /etc/hosts
```
```shell
192.168.197.10 broker
192.168.197.20 producer
192.168.197.30 consumer
```
- 확인
1. broker
```shell
#broker 터미널을 두개 만들어 동시에 해야한다.
/opt/kafka/bin/zookeeper-server-start.sh /opt/kafka/config/zookeeper.properties # 쥬키퍼 실행

/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties # 카프카 실행
```
![image](/assets/img/image/Pub_Sub패턴(kafka)/1.png)<br/>
다음과 같이 화면이 출력되면 카프카 서버와 주키퍼 서버가 연결이 성공적으로 된것이다.<br/>

- 토픽 생성하기
```shell
/opt/kafka/bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server broker:9092 # 토픽생성
/opt/kafka/bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server broker:9092  # 토픽 클러스터 확인
```
![image](/assets/img/image/Pub_Sub패턴(kafka)/2.png)<br/>


- 메세지 보내보기(producer)
```shell
/opt/kafka/bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server broker:9092   # producer의 콘솔이 하나 생성된다. 아무 거나 적고 엔터를 눌러 메세지를 카프카에 보내본다.
```
![image](/assets/img/image/Pub_Sub패턴(kafka)/3.png)<br/>

- 메세지 확인하기(consumer)
```shell
/opt/kafka/bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server broker:9092
```
![image](/assets/img/image/Pub_Sub패턴(kafka)/4.png)<br/>
다음과 같이 producer에 입력했던 메세지들을 확인 할 수 있다.<br/>

## 파이썬과 연결하기
```python
pip install kafka-python
```

`main.py`
```python
from kafka import KafkaProducer
import time

producer = KafkaProducer(
    bootstrap_servers=['192.168.197.10:9092']
)
start = time.time()

for i in range(100):
    producer.send('test', value="test".encode("utf-8"))
    producer.flush()

print("elapsed :", time.time() - start)
```

- 카프카 설정
```shell
vi /opt/kafka/config/server.properties
```
```shell
advertised.listeners=PLAINTEXT://192.168.197.10:9092    # 내 아이피로 수정 후 재시작
```
![image](/assets/img/image/Pub_Sub패턴(kafka)/5.png)<br/>

- 실행<br/>
![image](/assets/img/image/Pub_Sub패턴(kafka)/6.png)<br/>

## 장고 프로젝트와 연결하기
웹 프로젝트 실행<br/>
```shell
pip install kafka-python
```
`board/view/py` 추가
```python
from kafka import KafkaProducer
import time

producer = KafkaProducer(
    bootstrap_servers=['192.168.197.10:9092']
)


@login_required(login_url='/accounts/login')
def like(request, bid):
    post = Post.objects.get(id=bid)
    user = request.user

    producer.send('test', value="test".encode("utf-8")) #추가
    producer.flush()    #추가
    if post.like.filter(id=user.id).exists():
        post.like.remove(user)
        return JsonResponse({'message': 'deleted', 'like_cnt': post.like.count()})
    else:
        post.like.add(user)
        return JsonResponse({'message': 'added', 'like_cnt': post.like.count()})
```

- 확인<br/>
카프카 프로젝트에서 `consumer.py` 생성
```python
from kafka import KafkaConsumer
from json import loads

consumer = KafkaConsumer(
    'test',
    bootstrap_servers=['192.168.197.10:9092']
)

print('[begin] get consumer list')

for message in consumer:
    print("Topic: %s, Partition: %d, Offset: %d, Key: %s, Value: %s" % ( message.topic, message.partition, message.offset, message.key, message.value.decode('utf-8') ))

print('[end] get consumer list')
```
장고 프로젝트 실행 후 좋아요를 누르면<br/>
![image](/assets/img/image/Pub_Sub패턴(kafka)/7.png)<br/>
다음과 같이 `consumer.py` 콘솔에 문구가 전송된다.<br/>

### JSON형태로 받기
장고 프로젝트 `board/view/py` 변경
```python
from kafka import KafkaProducer
from json import dumps
import time

producer = KafkaProducer(
    acks=0,
    compression_type='gzip',
    bootstrap_servers=['192.168.197.10:9092'],
    value_serializer=lambda x: dumps(x).encode('utf-8')
)   # 추가

@login_required(login_url='/accounts/login')
def like(request, bid):
    post = Post.objects.get(id=bid) 
    user = request.user 

    data = data = { 'user' : user.id, 'post_id' : post.id } # 추가
    producer.send('logging.post.like', value= data)# 추가
    producer.flush()# 추가
    if post.like.filter(id=user.id).exists():
        post.like.remove(user)
        return JsonResponse({'message': 'deleted', 'like_cnt': post.like.count()})
    else:
        post.like.add(user)
        return JsonResponse({'message': 'added', 'like_cnt': post.like.count()})
```



- 확인<br/>
카프카 프로젝트에서 `consumer.py` 변경<br/>
```python
from kafka import KafkaConsumer
from json import loads

consumer = KafkaConsumer(
    'logging.post.like',    # 변경 후 실행
    bootstrap_servers=['192.168.197.10:9092']
)

print('[begin] get consumer list')

for message in consumer:
    print("Topic: %s, Partition: %d, Offset: %d, Key: %s, Value: %s" % ( message.topic, message.partition, message.offset, message.key, message.value.decode('utf-8') ))

print('[end] get consumer list')
```
![image](/assets/img/image/Pub_Sub패턴(kafka)/8.png)<br/>
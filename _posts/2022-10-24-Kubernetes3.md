---
title: '[Container] 쿠버네티스 - 모니터링'
categories:
    - Container
    - Docker
    - Kubernetes
    - Monitoring
tag:
    - Container
    - Docker
    - Kubernetes
    - Monitoring


last_modified_at: 2022-10-20T09:00:00+09:00
toc: true
---


# Helm
## Helm 설치 (master)
### 방화벽 해제
```shell
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
```

### 파일 다운 및 설치
```shell
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
## Helm을 이용해서 모니터링 환경 구축
### 네임스페이스 생성
```shell
kubectl create namespace monitoring
```
### Helm 레토지토리 추가
```shell
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```
### 레포지토리 업데이트
```shell
helm repo update
```

### 프로메테우스 + 그라파나 설치
```shell
wget https://raw.githubusercontent.com/grafana/helm-charts/main/charts/grafana/values.yaml
helm install prometheus prometheus-community/kube-prometheus-stack -f "values.yaml" --namespace monitoring
```

### 설치확인
![image](/assets/img/image/kubernetes3/1.png)<br/>
![image](/assets/img/image/kubernetes3/2.png)<br/>
프로미스 서비스의 설정을 바꿔준다.<br/>

![image](/assets/img/image/kubernetes3/3.png)<br/>
내 마스터ip:포트번호로 접속 확인한다.<br/>

### 그라파나 접속 확인<br/>
아이디: admin<br/>
비밀번호: prom-operator<br/>

### 그라파나 모니터링 탬플릿 불러오기
![image](/assets/img/image/kubernetes3/4.png)<br/>
![image](/assets/img/image/kubernetes3/5.png)<br/>
![image](/assets/img/image/kubernetes3/6.png)<br/>
![image](/assets/img/image/kubernetes3/7.png)<br/>


## 쿠버네티스 디플로이먼트를 이용한 아키텍쳐 만들기
### DB
#### 시크릿키
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-sec
  namespace: prod
data:
  MYSQL_ROOT_PASSWORD: cXdlcjEyMzQ=
```
#### pv,pvc
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv-prod
  namespace: prod
spec:
  capacity:
    storage: 10G
  accessModes:
  - ReadWriteOnce
  local:
    path: /mysql-volume
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - {key: kubernetes.io/hostname, operator: In, values: [node1]}
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10G
  storageClassName: ""
```


```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10G
  storageClassName: ""
```

#### 파드 생성
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
  labels:
    type: db
spec:
  nodeSelector:
    kubernetes.io/hostname: node1
  containers:
  - name: container
    image: mysql:latest
    ports:
    - containerPort: 3306
    envFrom:
    - secretRef:
        name: mysql-sec
    volumeMounts:
    - name: mysql-pvc-pv
      mountPath: /var/lib/mysql
  volumes:
  - name : mysql-pvc-pv
    persistentVolumeClaim:
      claimName: mysql-pvc
```

#### 서비스
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-svc
spec:
  selector:
    type: db
  ports:
  - port: 3306
    targetPort: 3306
```

### was
#### 파드 생성
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: tomcat-deployment-prod
spec:
 selector:
   matchLabels:
    type: was
 replicas: 2
 template:
   metadata:
     labels:
       type: was
   spec:
    containers:
    - name: container
      image: xc7230/tomcat:0.6
      ports:
      - containerPort: 8009
      resources:
        requests:
          cpu: 100m
        limits:
          cpu: 200m
```

#### 서비스
```yaml
apiVersion: v1
kind: Service
metadata:
  name: tomcat-svc
spec:
  selector:
    type: was
  ports:
  - port: 8009
    targetPort: 8009
```

### web
#### 파드 생성
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: httpd-cm-prod
data:
  TOMCAT_SVC_NAME: tomcat-svc-prod


apiVersion: apps/v1
kind: Deployment
metadata:
 name: httpd-deployment-prod
 namespace: prod
spec:
 selector:
   matchLabels:
    type: web
 replicas: 2
 template:
   metadata:
     labels:
       type: web
   spec:
    containers:
    - name: container
      image: xc7230/httpd:0.2
      ports:
      - containerPort: 80
      resources:
        requests:
          cpu: 100m
        limits:
          cpu: 200m
```

#### 서비스
```yaml
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  selector:
    type: web
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

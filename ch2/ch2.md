카프카 개발환경 구축
---

### ZooKeeper
- 주키퍼 : 분산 어플리케이션 관리를 위한 코디네이션 서비스
    - 스톰, NiFi, HBase 등에 사용됨.
    - 분산된 어플리케이션의 정보를 중앙에 집중하고 구성 관리, 네이밍, 동기화 서비스 제공
- 주키퍼는 서버를 앙상블로 구성하고 각각의 애플리케이션들이 클라이언트가 되어 서버와 연결함. 이 상태정보는 znode에 저장됨

### ZooKeeper 설치
```shell script
sudo apt-get update
sudo apt-get install openjdk-8-jdk
wget https://downloads.apache.org/zookeeper/zookeeper-3.6.0/apache-zookeeper-3.6.0.tar.gz
tar zxf apache-zookeeper-3.6.0.tar.gz
```
- symbolic link
```shell script
ln -s apache-zookeeper-3.6.0 zookeeper
ls -la zookeeper
```
- myid
``` shell script
mkdir data
echo 1 > /data/myid 환경설정
vi /usr/local/zookeeper/conf/zoo.cfg
```
- zoo.cfg
```shell script
trickTime=2000
initLimit=10
syncLimit=5
dataDir=/data
clientPort=2181
server.1=z1:2888:3888
server.2=z2:2888:3888
server.3=z3:2888:3888
server.4=z4:2888:3888
server.5=z5:2888:3888
```
- Zookeeper 실행/종료
```shell script
/usr/local/bin/zkServer.sh start
/usr/local/bin/zkServer.sh stop
```

### kafka 설치
```shell script
sudo apt-get update
sudo apt-get install openjdk-8-jdk
wget http://apache.mirror.cdnetworks.com/kafka/2.4.1/kafka_2.13-2.4.1.tgz
tar -xvzf kafka_2.13-2.4.1.tgz
```
- symbolic link
```shell script
ln -s kafka_2.13-2.4.1 kafka
ls -la kafka
```

- 데이터 경로 및 broker.id
```shell script
sudo mkdir -p /data1 
sudo mkdir -p /data2
```
- 환경설정
```shell script
sudo vim /usr/local/kafka/config/server.properties
```
- 아래내용수정
```shell script
broker.id=1
log.dirs=/data1,/data2
zookeeper.connect=z1:2181,z2:2181,z3:2181,z4:2181,z5:2181/kafka
```

- 아래내용추가(eng팀 ref.)
```shell script
delete.topic.enable=true
listeners=PLAINTEXT://:9092
advertised.listeners=PLAINTEXT://k1:9092
offsets.topic.replication.factor=1
```

- kafka 실행/종료
```
/usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties (-daemon)
/usr/local/kafka/bin/kafka-server-stop.sh
```

### 카프카 상태 확인
- TCP 포트 확인
```
netstat -ntlp | grep 2181
netstat -ntlp | grep 9092
```
- 주키퍼 지노드 이용한 카프카 정보 확인
```
/usr/local/zookeeper/bin/zkCli.sh
```
- 카프카 로그 확인
```
cat /usr/local/kafka/logs/server.log
```

### 카프카 시작하기(토픽 만들기)
- 카프카 토픽 생성
```
/usr/local/kafka/bin/kafka-topic.sh \
--zookeeper z1:2181,z2:2181,z3:2181,z4:2181,z5:2181/kafka \
--replication-factor 1 --partitions 1 --topic prac-topic --create
```
- 카프카 토픽 삭제
```
/usr/local/kafka/bin/kafka-topic.sh \
--zookeeper z1:2181,z2:2181,z3:2181,z4:2181,z5:2181/kafka \
--topic prac-topic --delete
```
- 메시지 퍼블리셔 작성
```
/usr/local/kafka/bin/kafka-console-producer.sh \
--broker-list k1:9092,k2:9092,k3:9092,k4:9092,k5:9092 \
--topic prac-topic
```
- 메시지 컨슈머 작성
```
/usr/local/kafka/bin/kafka-console-consumer.sh \
--broker-list k1:9092,k2:9092,k3:9092,k4:9092,k5:9092 \
--topic prac-topic --from-beginning
```

### 추가할것
- systemd 등록해서 해보기.. 오류가..

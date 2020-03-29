카프카 프로듀서
---
카프카의 토픽으로 메시지를 보내는 역할. 


콘솔 프로듀서로 메시지 보내기
--- 
- 카프카 토픽의 경우 미리 생성하지 않아도 사용할 수 있음
```shell script
# 토픽 자동 생성 옵션
auto.create.topic.enalbe = true
``` 
- 하지만, 어떤 목적으로 사용하는지 토픽에 대한 구분이 필요한 경우가 많기 떄문에 미리 생성하는 것이 나은 경우가 많다.

```shell script
# 카프카 토픽 생성
/usr/local/kafka/bin/kafka-topics.sh \
--zookeeper z1:2181,z2:2181,z3:2181,z4:2181,z5:2181/kafka \
--topic producer-test --partitions 1 --replication-factor 2 --create
```

- 콘솔 프로듀서 실행
    - 추가 옵션 등을 넣어 실행할 수 있음
```shell script
# 카프카 토픽 생성
/usr/local/kafka/bin/kafka-producer.sh \
--broker-list k1:9092, k2:9092, k3:9092, k4:9092, k5:9092\
--topic producer-test \
--compression-codec 'gzip' \
--request-required-acks 1
```

파이썬(자바) 프로듀서 
---

```python
from kafka import KafkaProducer

producer = KafkaProducer(acks=1, compression_type='gzip',
                         bootstrap_servers='k1:9092,k2:9092,k3:9092,k4:9092,k5:9092')
producer.send('producer-test', 'Apache Kafka is a distributed streaming platform')
```

```java
나중에 공부하고 추가하기
```

카프카 프로듀서 주요 옵션
---
* [Reference: Kafka Producer Configuration Documentation](https://kafka.apache.org/documentation/#producerconfigs)
* [Reference: PyKafka Producer Documentation](https://pykafka.readthedocs.io/en/latest/api/producer.html)
- bootstrap.servers 
    - 카프카 클러스터의 경우 마스터 노드와 같은 개념이 없기 때문에 모든 서버가 요청을 받을 수 있다. 
    - 장애 상황에 대해 대처하기 위해 호스트 서버 전체를 입력하는 것이 좋다.
- acks
    - 동기식 or 비동기식을 선택하는 기준. 메시지를 보낸 후 요청을 완료하기 전 승인의 수를 말함. 
        - 0 : 응답을 기다리지 않음
        - 1 : 리더의 데이터는 기록하지만 팔로워에 대한 기록을 하지 않음.
        - all(-1) : ISR 내의 모든 팔로워에 대한 응답을 기다림. (가장 강력한 데이터 무결성 보장)
    - 속도, 데이터의 무결성에 대한 Trade Off 관계가 있음.
- buffer.memory
    - 카프카 서버로 데이터를 보내기 위해 대기할 수 있는 메모리 바이트 크기
- compression.type 
    - 데이터를 압축해서 전송할 수 있는데, 어떤 방식으로 압축하맂 여부
- retries 
    - 오류상황 시 재전송 횟수
- batch.size 
    - 여러 데이터를 배치방식으로 전송할 때 해당 배치 크기를 바이트 단위로 지정할 수 있음. 
- linger.ms
    - 배치 크기 바이트에 도달하지 못했을 때 다른 메시지를 기다리는 최대 시간.
- max.request.size
    - 메시지의 최대 바이트 사이즈.(기본값 MB)
    
카프카 프로듀서의 메시지 전송 방법 (a.k.a acks) 
---
1. 메시지 손실가능성 Up But 전송 속도 Up : acks = 0 설정, 장애환경에서 문제발생
2. 메시지 손실가능성 적정 / 전송 속도 적정 : acks = 1 설정, 리더의 메시지만 응답확인
    - 일반적으로 메시지 손실이 나기 어렵지만, 파티션의 리더가 존재하는 부분에서 장애가 발생하면 손실 가능성이 존재함.
3. 메시지 손실가능성에 대한 강력한 보장 : acks = all(-1), 리더 및 팔로워의 메시지를 모두 확인
    - acks=all 옵션의 경우 단순히 하나의 옵션만으로 보장되는 것이 아니라 추가적인 설정이 필요함
        - min.insync.replicas = 1 의 경우 리플리케이션의 조건이 최소 하나이므로 acks=1 과 같은 효과.
    - 카프카 공식적으로는 메시지의 무결성을 보장하는 옵션을 제공함
        - acks=all, min.insync.replicas=2, replication.factor=3
    - min.insync.replicas=3이 아닌 이유는, 적어도 하나의 리플리케이션에서 문제가 발생하면, 동기화가 이루어지지 않아 메시지 전송이 이루어지지 않는다.
    
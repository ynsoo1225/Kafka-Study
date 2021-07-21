# [실습] kafka cluster 구축하기

# 1. AWS EC2 발급

ℹ️ **인스턴스 스펙**

- 3개 인스턴스
- m5.large

    ![1](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/95563c3d-7a3f-4965-95c6-74fc9decaa8f/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210721%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210721T100218Z&X-Amz-Expires=86400&X-Amz-Signature=8ab4d9af313432aa671cb8335c9e92893915d16e99eb757e7195b159fbe2f9fe&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

- 고정 IP 할당 → 탄력적 IP 주소 할당

    [[AWS] 7.AWS Elastic IP (EIP) 고정 아이피 할당 하기](https://goddaehee.tistory.com/192)

- 스토리지 50 GiB

![2](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/37ad54d9-f425-4d48-a71b-e7f765c703d6/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210721%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210721T100222Z&X-Amz-Expires=86400&X-Amz-Signature=5279f54f608f1f6f368d5d85a429126617a5d1007719f88a70ea091d3f0abd49&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

인스턴스 생성 완료 !!

# 2. /etc/hosts 설정

- (인스턴스에 자바 설치)
- vi /etc/hosts 수정

    ```bash
    // kafka1의 경우
    0.0.0.0   kafka1
    X.X.X.X   kafka2
    Y.Y.Y.Y   kafka3
    ```

# 3. Kafka 설치

```bash
$ wget https://archive.apache.org/dist/kafka/2.5.0/kafka_2.12-2.5.0.tgz
$ tar xvf kafka_2.12-2.5.0.tgz
$ ll
$ cd kafka_2.12-2.5.0
```

# 4. Zookeeper 설정

```bash
$ vi config/zookeeper.properties
# 아래 캡처처럼 설정
```

![3](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/cff5c8d3-98ab-457a-a156-e1e0570beb66/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210721%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210721T100230Z&X-Amz-Expires=86400&X-Amz-Signature=ef1cd99a0ce446ea1cb2b942ede86c2c73c393ee2271661009362cf89776788b&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

```bash
# 위에서 설정한 주키퍼 디렉토리 생성
$ mkdir /tmp/zookeeper
# 각 서버마다 주키퍼 id 부여
# kafka1은 1, kafka2는 2, kafka3는 3 .. 
$ echo 1 > /tmp/zookeeper/myid
```

# 5. Kafka 설정

```bash
$ vi config/server.properties

#server.properties
# 브로커 1~3까지 아이디 및 advertised.listeners 적절히 수정, 그 외에 나머지는 모두 동일함
broker.id=1
listeners=PLAINTEXT://:9092
advertised.listeners=PLAINTEXT://kafka1:9092
zookeeper.connect=kafka1:2181,kafka2:2181,kafka3:2181
```

# 6. 주키퍼 실행

```bash
# start
$ bin/zookeeper-server-start.sh -daemon config/zookeeper.properties

# stop
$ bin/zookeeper-server.stop.sh
```

반드시 주키퍼부터 실행해야 한다. 주키퍼가 카프카의 메타데이터를 관리하기 때문이다.

각 서버에서 주키퍼를 실행시켜 준다. 

# 7. 카프카 실행

```bash
# start
bin/kafka-server-start.sh -daemon config/server.properties

# stop
bin/kafka-server-stop.sh
```

각 서버에서 카프카를 실행시켜 준다. 

# 8. console-producer,consumer 테스트

```bash
[로컬]
# 테스트 편의를 위한 hosts 설정
$ vi /etc/hosts
{브로커 ip 주소} kafka1
{브로커 ip 주소} kafka2
{브로커 ip 주소} kafka3

# 토픽 생성
$ ./bin/kafka-topics.sh --create --zookeeper kafka1:2181,kafka2:2181,kafka3:2181 \
--replication-factor 3 --partitions 1 --topic test

# console-producer와 console-consumer를 동시에 켜고 topic에 데이터가 정상적으로 처리되는지 확인
$ ./bin/kafka-console-producer.sh --broker-list kafka1:9092,kafka2:9092,kafka3:9092 \
--topic test
> hi
> hello

$ ./bin/kafka-console-consumer.sh --bootstrap-server kafka1:9092,kafka2:9092,kafka3:9092 \
--topic test --from-beginning
hi
hello

# 토픽 리스트 확인
$ bin/kafka-topics.sh --list --bootstrap-server kafka1:9092,kafka2:9092,kafka3:9092
test
```

![4](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/47834e35-f360-44ce-8a7c-7ba39bcce043/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210721%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210721T100244Z&X-Amz-Expires=86400&X-Amz-Signature=a1d6cada233462854a1c9ee257f321e9b61930b00f91adfcc02224bb569f48ad&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

로컬에서 확인해 본 결과~

# References

[AWS에 카프카 클러스터 설치하기(ec2, 3 brokers)](https://blog.voidmainvoid.net/325)

[[kafka] 카프카 클러스터 구축](https://jinyes-tistory.tistory.com/242)
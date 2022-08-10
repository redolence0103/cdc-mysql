## package download
- https://www.confluent.io/hub/confluentinc/kafka-connect-jdbc
- https://debezium.io/releases/1.5/
- https://mvnrepository.com/artifact/mysql/mysql-connector-java/8.0.27
## guide (No suitable driver found)
- https://docs.confluent.io/kafka-connect-jdbc/current/index.html
## configuration 상세
- https://docs.confluent.io/kafka-connect-jdbc/current/sink-connector/sink_config_options.html
- https://docs.confluent.io/platform/current/connect/transforms/regexrouter.html
- https://debezium.io/blog/2017/09/25/streaming-to-another-database/
- https://docs.confluent.io/platform/current/connect/transforms/timestampconverter.html
## kafka 명령어 모음
- https://docs.confluent.io/platform/current/connect/references/restapi.html
- https://developer.confluent.io/learn-kafka/kafka-connect/rest-api/

## connector package
```bash
docker cp debezium-connector-mysql-1.5.4.Final-plugin.tar.gz kafka:/opt/kafka_2.13-2.8.1/connectors/debezium-connector-mysql-1.5.4.Final-plugin.tar.gz
docker cp confluentinc-kafka-connect-jdbc-10.5.1.zip kafka:/opt/kafka_2.13-2.8.1/connectors/
docker cp mysql-connector-java-8.0.27.jar kafka:/opt/kafka_2.13-2.8.1/connectors/
```

## docker 접속
```bash
docker exec -it kafka sh
```
## 해당 파일 압축 해제 및 이동
```bash
cd /opt/kafka_2.13-2.8.1/connectors

** 컨테이너에 zip 명령어가 없을때는 호스트환경에서 압축을 푼다음 tar로 묶어서 upload (압축 해제된 디렉토리에서 예: tar czf ../connector.tgz .)
unzip confluentinc-kafka-connect-jdbc-10.2.5.zip
tar xzf debezium-connector-mysql-1.5.4.Final-plugin.tar.gz
cp  mysql-connector-java-8.0.27.jar /opt/kafka_2.13-2.8.1/connectors/confluentinc-kafka-connect-jdbc-10.2.5
```
## /opt/kafka/config/connect-distributed.properties 파일의 plugin 경로를 수정
```bash
sed -i "/#plugin.path=/c\plugin.path=\/opt\/kafka_2.13-2.8.1\/connectors" /opt/kafka/config/connect-distributed.properties
```

## docker 재실행 (호스트 shell)
```bash
docker restart kafka
```

## docker 접속
```bash
docker exec -it kafka sh
```

## Distributed Mode로 kafka connect 실행
```bash
/opt/kafka/bin/connect-distributed.sh /opt/kafka/config/connect-distributed.properties
```

## connector 생성 확인
```bash
curl http://localhost:8083/
```

## connector를 생성하기 앞서 설치된 플러그인 목록을 조회한다.
```bash
curl --location --request GET 'localhost:8083/connector-plugins'
```

## Rest API 로 source connector 생성
```bash
curl --location --request POST 'http://localhost:8083/connectors' \
--header 'Content-Type: application/json' \
--data-raw '{
  "name": "source-test-connector",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "tasks.max": "1",
    "database.hostname": "mysql",
    "database.port": "3306",
    "database.user": "mysqluser",
    "database.password": "mysqlpw",
    "database.server.id": "184054",
    "database.server.name": "dbserver1",
    "database.allowPublicKeyRetrieval": "true",
    "database.include.list": "testdb",
    "database.history.kafka.bootstrap.servers": "kafka:9092",
    "database.history.kafka.topic": "dbhistory.testdb",
    "key.converter": "org.apache.kafka.connect.json.JsonConverter",
    "key.converter.schemas.enable": "true",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter.schemas.enable": "true",
    "transforms": "unwrap,addTopicPrefix",
    "transforms.unwrap.type": "io.debezium.transforms.ExtractNewRecordState",
    "transforms.addTopicPrefix.type":"org.apache.kafka.connect.transforms.RegexRouter",
    "transforms.addTopicPrefix.regex":"(.*)",
    "transforms.addTopicPrefix.replacement":"$1"
  }
}'
```

## Rest API 로 sink connector 생성
```bash
curl --location --request POST 'http://localhost:8083/connectors' \
--header 'Content-Type: application/json' \
--data-raw '{
  "name": "sink-test-connector",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
    "tasks.max": "1",
    "connection.url": "jdbc:mysql://mysql-sink:3306/sinkdb?user=mysqluser&password=mysqlpw",
    "auto.create": "false",
    "auto.evolve": "false",
    "delete.enabled": "true",
    "insert.mode": "upsert",
    "pk.mode": "record_key",
    "table.name.format":"${topic}",
    "tombstones.on.delete": "true",
    "connection.user": "mysqluser",
    "connection.password": "mysqlpw",
    "topics.regex": "dbserver1.testdb.(.*)",
    "key.converter": "org.apache.kafka.connect.json.JsonConverter",
    "key.converter.schemas.enable": "true",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter.schemas.enable": "true",
    "transforms": "unwrap, route, TimestampConverter",
    "transforms.unwrap.type": "io.debezium.transforms.ExtractNewRecordState",
    "transforms.unwrap.drop.tombstones": "true",
    "transforms.route.type": "org.apache.kafka.connect.transforms.RegexRouter",
    "transforms.route.regex": "([^.]+)\\.([^.]+)\\.([^.]+)",
    "transforms.route.replacement": "$3",
    "transforms.TimestampConverter.type": "org.apache.kafka.connect.transforms.TimestampConverter$Value",
    "transforms.TimestampConverter.format": "yyyy-MM-dd HH:mm:ss",
    "transforms.TimestampConverter.target.type": "Timestamp",
    "transforms.TimestampConverter.field": "update_date"
  }
}'
```

## Distributed Mode로 kafka connect 실행
```bash
curl --location --request GET 'http://localhost:8083/connectors'
curl --location --request GET 'http://localhost:8083/connectors/source-test-connector/config' --header 'Content-Type: application/json'
curl --location --request GET 'http://localhost:8083/connectors/sink-test-connector/config' --header 'Content-Type: application/json'
```

## connector 삭제
```bash
curl --location --request DELETE 'http://localhost:8083/connectors/source-test-connector'
curl --location --request DELETE 'http://localhost:8083/connectors/sink-test-connector'
```

## kafka Topic 목록확인 (cli)
```bash
kafka-topics.sh --list --bootstrap-server localhost:9092
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic dbserver1.testdb.accounts --from-beginning
```

## kafka connector API 요약
```bash
GET /connectors – returns a list with all connectors in use
GET /connectors/{name} – returns details about a specific connector
POST /connectors – creates a new connector; the request body should be a JSON object containing a string name field and an object config field with the connector configuration parameters
GET /connectors/{name}/status – returns the current status of the connector – including if it is running, failed or paused – which worker it is assigned to, error information if it has failed, and the state of all its tasks
DELETE /connectors/{name} – deletes a connector, gracefully stopping all tasks and deleting its configuration
GET /connector-plugins – returns a list of connector plugins installed in the Kafka Connect cluster
```

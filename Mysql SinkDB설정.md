## Mysql SINK-DB 설정
```bash
docker exec -it mysql-sink bash

mysql -u -p {password}

create database sinkdb;

use sinkdb;

CREATE TABLE accounts (
   account_id VARCHAR(255),
   role_id VARCHAR(255),
   user_name VARCHAR(255),
   user_description VARCHAR(255),
   update_date DATETIME DEFAULT CURRENT_TIMESTAMP,
   PRIMARY KEY (account_id)
);

use mysql;

// mysqluser 가 추가 되어 있는지 확인
select host, user from user;

// mysqluser 없으면 생성
CREATE USER 'mysqluser'@'%' IDENTIFIED BY 'mysqlpw';
// mysqluser 에게 권한 부여
GRANT ALL PRIVILEGES ON *.* TO 'mysqluser'@'%';

FLUSH PRIVILEGES;

## SourceDB 및 table 생성
```bash
docker exec -it mysql bash

mysql -u -p {password}

create database testdb;

use testdb;

CREATE TABLE accounts (
   account_id VARCHAR(255),
   role_id VARCHAR(255),
   user_name VARCHAR(255),
   user_description VARCHAR(255),
   update_date DATETIME DEFAULT CURRENT_TIMESTAMP,
   PRIMARY KEY (account_id)
);
```
## SourceDB mysql 사용자 등록
```bash
use mysql;

// mysqluser 가 추가 되어 있는지 확인
select host, user from user;

// mysqluser 없으면 생성
CREATE USER 'mysqluser'@'%' IDENTIFIED BY 'mysqlpw';
// mysqluser 에게 권한 부여
GRANT ALL PRIVILEGES ON *.* TO 'mysqluser'@'%';

FLUSH PRIVILEGES;
```

## data 입력
```bash
INSERT INTO accounts VALUES ("123456", "111", "Susan Cooper", "God", "2021-08-16 10:11:12");
INSERT INTO accounts VALUES ("123457", "111", "Rick Ford", "mistakes", "2021-08-16 11:12:13");
INSERT INTO accounts VALUES ("123458", "999", "Bradley Fine", "face", "2021-08-16 12:13:14");
```

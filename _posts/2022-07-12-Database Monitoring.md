# 테스트용 데이터베이스 생성

PostgreSQL를 이용해 검토를 진행한다.

도커 컨테이너 생성

```sh
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=1234 --name postgres postgres:latest
```

PSQL 접속

```shell
docker exec -it postgres psql -U postgres 
```

> 유저/데이터베이스 생성후 `psql -U <user> -d <db_name>` 으로 사용 

User 생성
```sql
create user ted password '1234' superuser;
```

`\du` 커맨드로 생성된 role를 확인할 수 있다.

데이터베이스 생성

```sql
create create database mydb owner ted;
```

생성한 데이터베이스에 접속

```psql
\c mydb ted
```

> 상기 과정까지는 도커 컨테이너를 생성하는 과정에서 환경 변수를 추가하는 것으로 모두 설정 가능하므로, PostgreSQL 도커 사용과 관련된 내용을 추가로 확인하자.

# 작성중
테스트를 위한 데이터 추가.. 이거 어디서 구하냐??



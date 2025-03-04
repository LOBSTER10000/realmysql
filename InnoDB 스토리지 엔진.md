### InnoDB 스토리지 엔진 잠금
---
먼저 'InnoDB'는 MYSQL의 데이터베이스 엔진입니다.
* InnoDB스토리지 엔진의 특징
  - InnoDB 스토리지 엔진은 스토리지 엔진 내부에서 레코드 기반의 잠금 방식을 탑재하고 있다.
  - InnoDB 스토리지 엔진은 레코드 기반의 잠금 방식 때문에 MyISAM보다 훨씬 뛰어난 동시성 처리를 제공한다. 하지만 이원화된 잠금처리로 인해 잠금에 대한 MySQL
    명령어를 통해 접근하기가 까다롭다.
  - 구 버전 MySQL 서버에서는 InnoDB 잠금 정보를 전달할 수 있는 도구는 'lock_monitor'와 'show engine innodb status' 명령이 전부였으며, 이 또한
    거의 어셈블리 코드를 보는 것 같아서 이해하기가 상당히 어려웠다.
  - 최근 버전에서는 InnoDB는 트랜잭션과 잠금, 잠금 대기중인 트랜잭션의 목록을 조회할 수 있는 방법이 도입되었다.

```sql
  조회 방법은 MySQL서버의 information_schema DB에 존재하는 INNODB_TRX, INNODB_LOCKS, INNODB_LOCK_WAITS 테이블을 조인하면
  어떤 트랜잭션이 어떠한 잠금을 대기하고 있으며, 해당 잠금을 어느 트랜잭션이 가지고 있는지 확인 가능하고, 해당 잠금을 가지고 있는 클라이언트를
  찾아서 종료도 가능하다.  
```
  
  - Performance Schema를 이용하면 InnoDB 스토리지 엔진의 내부 잠금에 대한 모니터링 방식도 추가되었다.

```sql
select substring_index(name, '/'. 1) as category, count(*) from performance_schema.setup_instruments
group by substring_index(name, '/', 1);
```
다음과 같이 쿼리를 작성하면 대략적으로 어떠한 대분류 카테고리가 있는지 확인 할 수 있다.
---
### InnoDB 스토리지 엔진의 잠금
InnoDB에는 '레코드 락' 뿐만 아니라 레코드와 레코드 사이의 간격을 잠그는 '갭락'이라는 것이 존재한다.
그렇다면 '레코드 락'과 '갭락' 그리고 '넥스트 키락'에 대해서 보겠습니다.

### 레코드락(Record lock, Record only lock)
'레코드 락'이란 레코드 자체만을 잠그는 것을 의미한다. 다른 DBMS들의 '레코드 락'과 동일한 역할을 한다.

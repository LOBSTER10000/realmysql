### 실행계획
---
#### 실행 계획 확인
- MYSQL의 서버 실행 계획은 'DESC' 또는 'EXPLAIN' 명령으로 확인할 수 있다.
- Format 옵션을 통해서 실행 계획의 표시 방법을 JSON, TREE, TABLE 형태로 표시할 수 있다.
```sql
// JSON 표기 예시
EXPLAIN FORMAT = JSON
SELECT *
FROM employees e
	INNER JOIN salaries s ON s.emp_no = e.emp_no
WHERE first_name = 'ABC'
```

```json
// JSON 표기 결과 값
EXPLAIN : {
	"query_block" : {
		"select_id" : 1,
		"cost_info": {
			"query_cost": "2.40" 
		},
		"nested_loop": [
			{
				"table": {
					"table_name" : "e",
					"access_type" : "ref",
					"possible_keys": [
							"PRIMARY",
							"ix_firstname"
					],
					"key":"ix_firstname",
					"used_key_parts" : [
							"first_name"
					],
```
10.2.2 쿼리의 실행 시간 확인
- MySQL 8.0.18 버전부터 쿼리의 실행 계획과 단계별 소요된 시간 정보를 확인할 수 있는 'explain analyze' 기능이 추가되었다.
- 항상 결과를 Tree 형태로 보여준다
  
```sql
EXPLAIN ANALYZE
SELECT e.emp_no, avg(s.salary)
FROM employee e
	INNER JOIN salaries s ON s.emp_no = e.emp_no
		AND s.salary > 500000
		AND s.from_date > '1990-01-01'
		AND s.to_date > '1990-01-01'
WHERE e.first_name = 'Matt'
GROUP BY e.hire_date 

A) -> Table scan on <temporary> (actual time = 0.001 ..0.004 rows = 48 loops = 1)
B)  -> Aggregate using temporary table (actual time = 3.799 .. 3.803. rows=48 loops =1 )
C)   -> Nested loop inner join (cost = 685.24 rows = 135)
                         (actual time = 0.367..3.602 rows=48 loops=1)
D)     -> Index lookup on e using ix_firstname(first_name = 'Matt') (cost=215.08 rows=233)
E)     -> Filter: ((s.salry>50000) and (s.from_date <= DATE'1990-01-01')
						and (s.to_date > DATE '1990-01-01')) (cost = 0.98 rows = 1)
            (actual time = 0.009..0.011 rows=0 loops=233))
F)        -> Index lookup on s using PRIMARY (emp_no = e.emp_no) (cost=0.98 rows=10)
							(actual time = 0.007 .. 0.009 rows=10 loops=233)

```

- 들여쓰기는 호출 순서를 의미한다
   (1) 들여쓰기가 같은 레벨에서는 상단에 위치한 라인이 먼저 실행
   (2) 들여쓰기가 다른 레벨에서는 가장 안쪽에 위치한 라인이 먼저 실행
- actual time = 실제 소요된 시간, 레코드 건수 = rows, 반복 횟수 loops
   (3) actual time = 0.007 .. 0.009:
       - 첫번째 숫자 값은 첫번째 레코드를 가져오는데 걸린 평균 시간
       - 두번째 숫자 값은 마지막 레코드를 가져오는데 걸린 평균 시간

```sql
explain analyze 명령은 explain 명령과 달리 실행 계획만 추출하는 것이 아니라
실제 쿼리를 실행하고 사용된 실행 계획과 소요된 시간을 보여주는 것이다.
따라서 쿼리의 실행 시간이 아주 많이 걸리면 explain analyze를 사용하면
쿼리가 완료되야 실행 계획 결과를 확인하므로 explain으로 실행 계획을 확인해
어느정도 튜닝한 이후에 explain analyze 명령을 사용 하자
```

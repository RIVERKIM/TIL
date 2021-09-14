# My SQL 개념

생성일: 2021년 9월 14일 오후 2:07

## SQL 분류

### DML(Data Manipulation Language)

- 데이터 조작 언어
- 데이터를 조작(선택, 삽입, 수정 삭제) 하는데 사용하는 언어
- DML 구문이 사용되는 대상은 테이블의 행
- DML 사용하기 위해서는 꼭 그 이전에 테이블이 정의되어 있어야함.
- SQL문 중 SELECT, INSERT, UPDATE, DELETE가 이 구문에 해당
- 트랜잭션(transaction)이 발생하는 SQL도 이 DML에 속함.

### DDL(Data Definition Language)

- 데이터 정의 언어
- 데이터베이스, 테이블, 뷰, 인덱스 등의 데이터베이스 개체를 생성/ 삭제 / 변경하는 역할
- CREATE, DROP, ALTER구문
- DDL은 트랜잭션 발생시키지 않음
- ROLLBACK이나 COMMIT 사용불가
- DDL문은 실행 즉시 MySql에 적용.

### DCL(Data Control Language)

- 데이터 제어 언어
- 사용자에게 어떤 권한을 부여하거나 빼앗을 때 주로 사용하는 구문
- GRANT REVOKE 등

## 기본 명령어들

- Use: 사용할 데이터베이스 지정.

```jsx
USE world;
```

- SHOW TABLE

```jsx
SHOW TABLE <status>
```

- DESCRIBE

```jsx
DESCRIBE <TABLE>
DESC <TABLE>
```

- SELECT

```jsx
SELECT <select_expr>
[FROM table_references]
[WHERE where_references]
[GROUP BY {col_name | expr | position}]
[HAVING where_condition]
[ORDER BY {col_name | expr | position}]
```

- 관계 연산자

```jsx
NOT, AND, OR,
BETWEEN ~ AND ~;
IN('')
//(%무엇이든, _ 한글자)
LIKE
```

- Sub Query

```jsx
select *
from city
where CountryCode = (select CountryCode
from city
where Name='Seoul');
```

- Any, ALL , SOME

```jsx
//ANY = SOME서브 쿼리의 결과중 하나만 만족해서 가능
//ALL 결과가 모두 만족.
select *
from city
where Population > <> ( select Population from city
where Distinct= 'New York');
```

- ORDER BY

```jsx
//결과가 출력되는 순서 조절 
// 기본적으로 오름차순, 뒤에 DESC 내림차순
select *
from city
ORDER BY CountryCode ASC, Population DESC;
```

- DISTINCT

```jsx
//중복된 것을 제거. 테이블 크기가 클수록 효율적
SELECT distinct CountryCode
from city;
```

- LIMIT

```jsx
// 출력 개수를 제한, 처리량이 많을 때 성능 개선
SELECT *
FROM city
ORDER BY Population DESC
LIMIT 10;
```

- Group By

```jsx
// 그룹으로 묶어주는 역할, 집계 함수(aggregate function)를 함께 사용.
// AVG(), MIN(), MAX(), COUNT(), COUNT(DISTINCT), STDEV(), VARIANCE()

SELECT CountryCode, MAX(Population) as 'Population'
FROM city
GROUP BY CountryCode;
```

- HAVING

```jsx
//WHERE와 비슷한 개념으로 조건 제한
// 집계 함수에 대해서 조건을 제한하는 편리한 개념
// HAVING 절은 반드시 GROUP BY 절 다음에 나와야함.
SELECT CountryCode, MAX(Population) as 'Population'
FROM city
GROUP BY CountryCode
HAVING MAX(Population) > 800000;
```

- ROLLUP

```jsx
//총합 또는 중간합계가 필요한 경우
// GROUP BY절과 함께 WITH ROLLUP 문 사용.

SELECT CountryCode, Name, SUM(Population)
FROM city
GROUP BY CountryCode, Name WITH ROLLUP;

```

- JOIN

```jsx
// 데이터베이스 내의 여러 테이블에서 가져온 레코드를 조합하여
// 하나의 테이블이나 결과집합으로 표현.
//ON 대신 where 가능
//INNER JOIN, ON에 조건절
SELECT *
FROM city JOIN Country ON city.CountryCode = country.Code;

//NATURAL JOIN
//두 테이블에 가은 이름의 열이 있을때만 동작. ON 필요없음.
SELECT *
FROM city NATURAL JOIN Country;

//OUTER JOIN
SELECT *
FROM city <LEFT, RIGHT> OUTER JOIN country
ON city.CountryCode = country.Code;

//CROSS JOIN
SELECT *
FROM city CROSS JOIN country

```

### MySQL 내장 함수

```jsx
//LENGTH()
SELECT LENGTH('DFDFDF');

//CONCAT()
SELECT CONCAT('A', 'A')

//LOCATE()위치 반환, 단 MYSQL은 1부터 인덱스
SELECT LOCATE('ABC') 

//LEFT(), RIGHT()
SELECT LEFT("ADSFSAF",5); //왼쪽에서 5개만 뽑음.

//LOWER(), UPPER()
SELECT LOWER("dfdfd");

//REPLACE()
SELECT REPLACE('MYSQL', 'MY', 'MS');

//TRIM()
SELECT TRIM('         DDF        ');//DDF

//FLOOR(), CEIL(), ROUND()
SELECT FLOOR(33.3);

//SQRT(), POW(), EXP(), LOG(), SIN(), COS(), TAN()

//ABS(), RAND() // 0.0 ~ 1.0
SELECT ROUND(RAND() * 100, 0);

//NOW(), CURDATE(), CURTIME(),
SELECT NOW();
//DATE(), MONTH(), DAY() HOUR(), MINUTE(), SECOND()
SELECT MONTH(NOW());
//MONTHNAME(), DAYNAME()
SELECT MONTHNAME(NOW());
```

## SQL 고급

- CREATE TABLE  AS SELECT

```jsx
//CITY 테이블과 똑같은 CITY2 테이블 생성.
CREATE TABLE city2 AS SELECT * FROM CITY;
```

- CREATE DATABASE

```jsx
CREATE DATABASE aa;

USE aa;
//GUI 로도 생성 가능
```

- CREATE TABLE

```jsx
CREATE TABLE test2 (
	id INT NOT NULL PRIMARY EKY,
	col1 INT NULL,
	col2 VARCHAR(45) NULL
);
```

- ALTER TABLE

```jsx
ALTER TABLE test2
ADD col4 INT NULL
MODIFY col2 VARCHAR(20) NULL;
DROP col2;

```

### Index

- 테이블에서 원하는 데이터를 빨리 찾기 위해 사용
- 일반적으로 데이터 검색할 때 순서대로 테이블 전체를 검색하므로 데이터가 많아지면 탐색하는 시간이 늘어남
- 검색과 질의를 할 때 테이블 전체를 읽지 않기 때문에 빠름
- 설정된 컬럼 값을 포함한 데이터의 삽입 삭제, 수정 작업이 원본 테이블에서 이루어질경우, 인덱스도 함께 수정되어야 함.
- 인덱스가 있는 테이블은 처리속도가 느려질 수 있으므로 수정보다는 검색이 자주 사용되는 테이블에 사용하는 것이 좋음.

```jsx
//test 테이블 co1컬럼에 인덱스 생성
CREATE INDEX ColIdx
ON test (col1)
//인덱스 정보 보기
SHOW INDEX FROM test;
//중복 값을 허용하지 않는 인덱스
CREATE UNIQUE INDEX Col2Index ON test (co2);

//FULLTEXT INDEX일반적인 인덱스와는 달리 매우 빠르게 테이블의 모든 텍스트 컬럼을 검색
ALTER TABLE test
ADD FULLTEXT ColIdx3(col3);

// INDEX 삭제
ALTER TABLE test
DROP INDEX ColIdx3

DROP INDEX Col2Idx ON test;
```

### VIEW

- 데이터베이스에 존재하는 일종의 가상 테이블
- 실제 테이블처럼 행과 열을 가지고 있지만 실제로 데이터를 저장하지 않음.
- MYSQL에서 뷰는 다른 테이블이나 다른 뷰에 저장되어 있는 데이터를 보여주는 역할만 수행.
- 뷰를 사용하면 여러 테이블이나 뷰를 하나의 테이블처럼 사용할 수 있음.
- Index를 가질 수 없다.
- 특정 사용자에게 테이블 전체가 아닌 필요한 컬럼만 보여줄 수 있음.
- 복잡한 쿼리를 단순화해서 사용
- 한번 정의된 뷰는 변경 불가능.
- 삽입, 삭제, 갱신 작업에 많은 제한 사항을 가짐.
- 자신만의 인덱스를 가질 수 없음.

```jsx
//VIEW 생성
CREATE VIEW testView AS
SELECT col1, col2
FROM test;

//VIEW 수정
ALTER VIEW testView AS
SELECT col1, col2, col3
FROM test;

//VIEW 삭제
DROP VIEW testView
```

### Table 변경

- INSERT

```jsx
INSERT INTO test
VALUES('', '', '');

INSERT INTO test2 SELECT * FROM test;
```

- UPDATE

```jsx
UPDATE test
SET col1 = 1, col2 = 3
WHERE id = 1;

```

- DELETE

```jsx
DELETE FROM test
WHERE id = 1;

//지웠지만 복구가능.

```

- TRUNCATE

```jsx
// 데이터 전부 삭제, 복구 불가.
TRUNCATE TABLE test;

```

- DROP TABLEQ

```jsx
DROP TABLE
DROP DATABASE
```
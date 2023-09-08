## 06장 SELECT 구문

### WHERE 구

- IN으로 OR 조건을 간단하게 작성

```sql
SELECT name, address 
FROM Address 
WHERE address = '서울시' 
  OR address = '화성시' 
  OR address = '수원시';
```

```sql
SELECT name, address 
FROM Address 
WHERE address IN ('서울시', '화성시', '수원시');
```

<br>

### 뷰와 서브쿼리

- 뷰
    
    > SELECT 구문을 DB 안에 저장해둠 (**내부에 데이터를 보유하지 않고 SELECT 구문만을 저장**)
    > 
    - 뷰 만드는 방법
    
    ```sql
    CREATE VIEW [뷰이름]([필드이름1], [필드이름2], ...) AS
    ```
    
    ```sql
    CREATE VIEW CountAddress (v_address, cnt)
    AS
      SELECT address, COUNT(*)
      FROM Address
      GROUP BY address;
    ```
    
    - 사용 시 **테이블 대신 뷰를 FROM 구에 지정**
    
    ```sql
    SELECT v_address, cnt 
    FROM CountAddress;
    ```
    
    - 즉, `추가적인 SELECT 구문을 실행`하는 **중첩 구조**를 가진다.
    
    ```sql
    SELECT v_address, cnt
    FROM (SELECT address AS v_address, COUNT(*) AS cnt
          FROM Address
          GROUP BY address) AS CountAddress;
    ```
    
- 서브쿼리
    - IN 내부에서 서브쿼리 사용
    - **SQL은 서브쿼리부터 실행**
    
    ```sql
    SELECT name
    FROM Address
    WHERE name IN (SELECT name FROM Address2);
    ```
    
    💡 IN에 **상수로 하드코딩**을 하면 테이블의 **데이터가 변할 때마다 SELECT 구문을 수정해야 한다.** 하지만 **서브쿼리를 사용**하면 IN 내부의 서브쿼리가 **SELECT 구문 전체가 실행될 때마다 다시 실행된다.** (`동적`)


<br>
<br>
<br>

---

<br>
<br>

## 07장 조건 분기, 집합 연산, 윈도우 함수, 갱신

### SQL과 조건 분기

- `CASE문`
- 강력한 점: 식이라는 것
    - 식을 적을 수 있는 곳이라면 어디든지 적을 수 있다.
    - SELECT, WHERE, GROUP BY, HAVING, ORDER BY
    - SQL 성능과도 밀접한 관련이 있으니 잘 기억하고 있자.

<br>

### SQL의 집합 연산

- `UNION`
    - 합집합을 구할 때 중복된 레코드를 제거
    - 만약 중복을 제거하고 싶지 않다면 `UNION ALL`로 ALL 옵션을 붙이면 된다.
- `INTERSECT`
    - 교집합을 구할 때 중복된 레코드를 제거
- `EXCEPT`
    - 차집합
    - `UNION`과 `INTERSECT`는 테이블 순서에 상관없이 결과가 동일하지만 `EXCEPT`은 결과가 달라진다.
    - 즉, **교환 법칙이 성립하지 않는다.**

<br>

### 윈도우 함수

- 특징 - **"집약 기능이 없는 GROUP BY 구"**
    - GROUP BY 구는 `자르기`와 `집약`이라는 두 개의 기능으로 구분
- `PARTITION BY`라는 구로 수행
    - 자른 후 집약하지 않으므로 **출력 결과의 레코드 수가 입력되는 테이블의 레코드 수와 같다.**
- 기본적인 구문
    - 작성하는 가능한 곳은 SELECT 구라고만 생각해도 문제 없다.
    
    ```sql
    집약 함수 OVER(PARTITION BY 키)
    ```
    
    ```sql
    SELECT address,
    			 COUNT(*) OVER(PARTITION BY address)
    FROM Address;
    ```
    
- COUNT / SUM 같은 일반 함수 이외에도, 윈도우 함수 전용 함수로 제공되는 `RANK` 또는 `ROW_NUMBER` 등의 순서 함수가 있다.
    
    ```sql
    SELECT name, age,
    			 RANK() OVER(ORDER BY age DESC) AS rnk
    FROM Address;
    ```
    
    만약, 같은 순위를 가지게 되었을 때 다음 순위를 건너뛰게 하기 싫다면 `DENSE_RANK` 함수 사용하면 된다.

## 08장 UNION을 사용한 쓸데없이 긴 표현

[ UNION ]
- `장점`: 큰 문제를 작은 문제로 나눌 수 있어 생각하기 쉽다는 점
- `단점`: 외부적으로 하나의 SQL 구문을 실행하는 것처럼 보이지만, **내부적으로는 여러 개의 SELECT 구문을 실행하는 실행 계획으로 해석되어 성능적인 측면에서 굉장히 큰 단점**을 가짐
- 따라서 SQL에서 조건 분기를 할 때 UNION을 사용해도 좋을지 여부는 신중히 검토해야 한다.

<br>

### [1] UNION을 사용한 조건 분기와 관련된 간단한 예제

- 2001년까지는 세금을 포함하지 않는 가격을 2002부터는 세금을 포함한 가격을 표시해야 한다.

```sql
SELECT item_name, year, price_tax_ex AS price
	FROM items
	WHERE year <= 2001
UNION ALL
SELECT item_name, year, price_tax_in AS price
	FROM items
	WHERE year >= 2002
```

이 코드는 문제를 가지고 있다. 1. 쓸데없이 길어 읽기 어려움 2. 두 개의 쿼리를 두 번이나 실행한다.

- UNION을 사용했을 때의 실행 계획 문제
    - 어떤 DB를 사용하더라도 `UNION 쿼리는 items 테이블에 2회 접근`한다.
    - 이 때마다 `TABEL ACCESS FULL이 발생`하므로 **읽어들이는 비용도 테이블의 크기에 따라 선형으로 증가한다.**
        - 데이터 캐시에 테이블의 데이터가 있으면 어느정도 증상이 완화되겠지만
        - **테이블의 크기가 커지면 캐시 히트율이 낮아지므로** 이런 것도 기대하기 힘들다.
- 정확한 판단 없는 UNION 사용 회피하자.

<br>

### [2] WHERE 구에서 조건 분기를 하는 사람은 초보자

SQL에 이런 격언이 있다. `조건 분기를 WHERE 구로 하는 사람들은 초보자다. 잘하는 사람은 SELECT 구만으로 조건 분기를 한다.`

- 다음과 같이 최적화가 가능하다.
    
    ```sql
    SELECT item_name, year,
    	CASE WHEN year <= 2001 THEN price_tax_ex
    			 WHEN year >= 2002 THEN price_tax_in
    	END AS price
    FROM items;
    ```
    
    - 테이블의 크기가 커질수록 UNION보다 성능이 훨씬 좋아진다.

<br>

### [3] SELECT 구를 사용한 조건 분기의 실행 계획

- Items 테이블에 대한 접근이 1회로 줄어든다.
    - 즉, 성능이 2배 좋아졌다.
- SQL문 가독성이 좋아졌다.

💡 **SQL 구문의 성능이 좋은지 나쁜지는 반드시 실행 계획 레벨에서 판단해야 한다.** SQL 구문에는 어떻게 데이터에 접근할지를 나타내는 접근 경로가 쓰여져 있기 않기 때문이다.

사실 이는 좋은게 아니다. “사용자가 데이터에 접근 경로라는 물리 레벨의 문제를 의식하지 않도록 하고 싶다”라는 것이 RDB와 SQL이 가진 컨셉이기 때문이다. 하지만, 이런 뜻을 이루기에는 현재의 RDB와 SQL는 역부족이다.

- UNION과 CASE 쿼리를 구문적인 관점에서 비교하면 재밌다.
    - `UNION`: SELECT `구문`을 기본 단위로 분기한다. (절차 지향적인 발상을 벗어나지 못한 것)
    - `CASE`: 문자 그대로 `식`을 바탕으로 사고한다.
    - `구문`에서 `식`으로 **사고를 변경하는 것은 SQL을 마스터하는 열쇠 중 하나**이다.

<br>
<br>
<br>

---

<br>

## 09장 집계와 조건 분기

- 인구 테이블 - Population

| prefecture (지역 이름) | sex (성별) | pop (인구) |
| --- | --- | --- |
| 성남 | 1 | 60 |
| 성남 | 2 | 40 |
| 수원 | 1 | 90 |
| 수원 | 2 | 100 |
| … | … | … |

### [1] 집계 대상으로 조건 분기

- UNION을 사용한 방법

  ```sql
  SELECT prefecture, SUM(pop_men) AS pop_men, SUM(pop_women) AS pop_wom
  	FROM (SELECT prefecture, pop AS pop_men, NULL AS pop_wom
  				FROM Population
  				WHERE sex = '1'
  			UNION
  				SELECT prefecture, NULL AS pop_men, pop AS pop_wom
  				FROM Population
  				WHERE sex = '2' ) TMP
  GROUP BY prefecture;
  ```

  - 위 쿼리의 가장 큰 문제는 `WHERE 구에서 sex 필드로 분기`하고, `결과를 UNION으로 머지`한다는 **절차 지향적인 구성에 있다.**

  - 위 UNION의 실행 계획
      - Population 테이블에 풀 스캔이 2회 수행된다.
- 집계의 조건 분기도 CASE 식을 사용
    
    ```sql
    SELECT prefecture,
    			 SUM(CASE WHEN sex = '1' THEN pop ELSE 0 END) AS pop_men,
    			 SUM(CASE WHEN sex = '2' THEN pop ELSE 0 END) AS pop_wom
    FROM Population
    GROUP BY prefecture;
    ```
    
- CASE 식의 실행 계획
    - `데이터의 풀 스캔`이 `1회`로 감소했다.
    - 성능면에서 더 좋은 힘을 발휘한다.

<br>

### [2] 집약 결과로 조건 분기

| emp_id | team_id | emp_name | team_name |
| --- | --- | --- | --- |
| 201 | 1 | Joe | 상품기획 |
| 201 | 2 | Joe | 개발 |
| 202 | 2 | Jim | 개발 |
| … | … | … | … |
- UNION을 사용한 조건 분기
    
    ```sql
    SELECT emp_name,
    			 MAX(team) AS team,
    FROM Employees
    GROUP BY emp_name
    HAVING COUNT(*) = 1
    UNION
    SELECT emp_name,
    			 '2개를 겸무' AS team,
    FROM Employees
    GROUP BY emp_name
    HAVING COUNT(*) = 2
    UNION
    SELECT emp_name,
    			 '3개 이상을 겸무' AS team,
    FROM Employees
    GROUP BY emp_name
    HAVING COUNT(*) >= 3;
    ```
    
    - 조건 분기가 레코드 값이 아닌 `집합의 레코드 수에 적용`된다. (`HAVING`) 하지만 **UNION으로 머지하고 있는 이상**, `구문 레벨의 분기`일 뿐이다. 즉, **WHERE 구를 사용할 때와 다르지 않다.**
    - 실행 계획: `3번` 테이블에 접근
- CASE 식을 사용한 조건 분기
    
    ```sql
    SELECT emp_name,
    			 CASE WHEN COUNT(*) = 1 THEN MAX(team)
    			      WHEN COUNT(*) = 2 THEN '2개를 겸무'
    			      WHEN COUNT(*) >= 3 THEN '3개 이상을 겸무'
    			 END AS team
    FRM Employees
    GROUP BY emp_name;
    ```
    
- CASE 식을 사용한 조건 분기의 실행 계획
    - 테이블에 접근 비용을 `3분의 1`로 줄일 수 있다.
    - [ 추가 ] GROUP BY의 HASH 연산도 `3회 → 1회`로 줄어든다.
    - 이게 가능했던 것은 `집약 결과(COUNT 함수의 리턴값)를 CASE 식의 입력으로 사용했기 때문`임.
        - 다르게 말하면 집약 함수의 결과가 스칼라 (더 이상 분할 불가능한 값)이 되는 것이다.

💡 **WHERE 구만 아닌 HAVING 구에서 조건 분기를 하는 사람도 초보자이다.**

<br>
<br>
<br>

---

<br>

## 10장 그래도 UNION이 필요한 경우

### [1] UNION을 사용할 수 밖에 없는 경우

- 머지 대상이 되는 `SELECT 구문들에서 사용하는 테이블이 다른 경우`가 대표적이다.

```sql
SELECT col_1
	FROM Table_A
	WHERE col_2 = 'A'
UNION ALL
SELECT col_3
	FROM Table_B
	WHERE col_4 = 'A'
```

- FROM 구에서 테이블을 결합하면 CASE 식을 사용해 원하는 결과를 구할 수 있다.
    - 하지만, 이렇게 하면 **필요 없는 결합이 발생해서 성능적으로 악영향이 발생**한다.

### [2] UNION을 사용하는 것이 성능적으로 더 좋은 경우

- UNION 이외에 다른 방법으로도 풀 수 있지만, UNION을 사용하는 편이 더 성능이 좋을 수 있다.
- UNION을 사용했을 때 `좋은 인덱스(압축을 잘 하는)`를 사용하지만, 이외의 경우에는 **테이블 풀 스캔이 발생한다면, UNION을 사용한 방법이 성능적으로 더 좋을 수 있다.**

<br>
<br>
<br>

---

<br>

## 11장 절차 지향형과 선언형

예외적인 몇 가지 방법을 제외하면 UNION을 사용하지 않는 것이 성능 / 가독성 면에서 좋다. 원래 **UNION은 조건 분기를 위해 만들어진 것이 아니므로** 당연한 결과이다.

### [1] 구문 기반과 식 기반

- SQL 초보자는 절차지향 세계에 살고 있다.
    - 기본 단위: `구문`
- SQL 중급자 이상
    - 기본 단위: `식`

💡 **SQL 기본적인 체계는 선언형이다. 이 세계의 주역은 구문이 아니라 식이다.**

- SQL 구문의 각 부분(SELECT, FROM, WHERE, GROUP BY, HAVING, ORDER BY)에 작성하는 것은 모두 식이다.
- 열이름 또는 상수만 기술하는 경우도 마찬가지이다.

> `SQL 구문 내부에는 식을 작성`하지, 구문을 작성하지 않는다.
> 

### [2] 선언형의 세계로 도약

절차 지향형 세계에서 선언형 세계로 도약하는 것이 곧 SQL 능력의 핵심이다.

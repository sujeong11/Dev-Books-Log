## 23강 레코드에 순번 붙이기

### [1] 기본 키가 한 개의 필드일 경우

- 체중 테이블 (Weights)
    
    
    | student_id | weight |
    | --- | --- |
    | A100 | 50 |
    | A101 | 55 |
    | ... | ... |
- 학생 ID를 오름차순으로 순번으로 붙여보자.
- **[1] 윈도우 함수 사용**
    
    ```sql
    SELECT student_id,
           ROW_NUMBER() OVER (ORDER BY student_id) AS seq
    FROM Weights;
    ```
    
    - 테이블 스캔: 1회
- **[2] 상관 서브쿼리 사용**
    - MySQL처럼 ROW_NUMBER 함수를 사용할 수 없을 때
    
    ```sql
    SELECT student_id,
           (SELECT COUNT(*)
            FROM Weights W2
            WHERE W2.student_id <= W1.student_id) AS seq
    FROM Weights W1;
    ```
    
    - 테이블 스캔: 2회 (W1, W2)

### [2] 기본 키가 여러 개의 필드로 구성되는 경우

- 체중 테이블2 (Weights2)
    
    
    | class | student_id | weight |
    | --- | --- | --- |
    | 1 | A100 | 50 |
    | 1 | A101 | 55 |
    | ... | ... | ... |
- **[1] 윈도우 함수 사용**
    
    ```sql
    SELECT class, student_id,
           ROW_NUMBER() OVER (ORDER BY class, student_id) AS seq
    FROM Weights2;
    ```
    
- **[2] 상관 서브쿼리 사용**
    - `다중 필드 비교`를 사용
    
    ```sql
    SELECT class, student_id,
           (SELECT COUNT(*)
            FROM Weights2 W2
            WHERE (W2.class, W2.student_id) 
                    <= (W1.class, W1.student_id)) AS seq
    FROM Weights2 W1;
    ```
    
    - **장점**
    - 필드 자료형을 원하는 대로 지정할 수 있다. 숫자와 문자열, 문자열 숫자라도 가능하다.
    - 필드가 늘어나도 확장이 간단하다.

### [3] 그룹마다 순번을 붙이는 경우

- 학급마다 순번을 붙여보자.
- **[1] 윈도우 함수 사용**
    
    ```sql
    SELECT class, student_id,
           ROW_NUMBER() OVER (PARTITION BY class ORDER BY student_id) AS seq
    FROM Weights2;
    ```
    
- **[2] 상관 서브쿼리 사용**
    
    ```sql
    SELECT class, student_id,
           (SELECT COUNT(*)
            FROM Weights2 W2
            WHERE W2.class = W1.class
              AND W2.student_id <= W1.student_id AS seq
    FROM Weights2 W1;
    ```
    

### [4] 순번과 갱신

- 검색이 아닌 갱신에서 순번을 매기는 방법을 알아보자.
- 체중 테이블3 (Weights3)
    
    
    | class | student_id | weight | seq |
    | --- | --- | --- | --- |
    | 1 | A100 | 50 |  |
    | 1 | A101 | 55 |  |
    | ... | ... | ... | |
- **[1] 윈도우 함수 사용**
    - 서브 쿼리도 함께 사용해야 한다.
    
    ```sql
    UPDATE Weight3
      SET seq = (SELECT seq
                 FROM (SELECT class, student_id,
                          ROW_NUMBER()
                              OVER (PARTITION BY class 
                                        ORDER BY student_id) AS seq
                       FROM Weight3) SeqTb1
                 WHERE Weight3.class = SeqTb1.class
                   AND Weight3.student_id = SeqTb1.student_id);
    ```
    
- **[2] 상관 서브쿼리 사용**
    
    ```sql
    UPDATE Weight3
    	SET seq = (SELECT COUNT(*)
                 FROM Weight3 W2
                 WHERE W2.class = Weight3.class
                   AND W2.student_id <= Weight3.student_id);
    ```

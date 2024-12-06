문장의 마지막에는 ; 를 쓴다 
예약어 : SQL 안에서 정해진 역할을 하는 단어.
예약어는 대문자로 적는게 국룰
단어 사이는 공백으로 구분
하지만  , ; 사이에 공백은 상관이 없다 
# 문법순서 
**SELECT > DISTINCT > FROM** >**WHERE** >**GROUP BY** >**HAVING** >**ORDER BY**
# 실행순서
**FROM**  > ON > JOIN > **WHERE** > **GROUP BY** > **HAVING** >**SELECT** > DISTINCT >**ORDER BY**
테이블에서 -> 전체조건 -> 그룹화 -> 그룹조건 -> 가져올 열 선택 -> 중복제거 -> 정렬 
## mysql
> <=> null 값도 비교 가능 , = 

# SELECT 
* '무엇을' 

```SQL
SELECT column_name 
SELECT column_name1,column_name2 // 여러개 선택 가능 , 불려온 순서대로 가져옴 
SELECT * // 모든 컬럼 가져오기  
SELECT column_name AS cd // 별명 지정 
```

# FROM
* '어디에서'

```SQL
FROM table_name;
```

# WHERE
* 조건주기 
```SQL
WHERE member_id = 2 // member_id 가 2인경우 
```
* 비교연산자 
 ㅜ 예시 :> < >= <= != =(<-> : NULL 비교 가능, mysql)
결과 : 1(TRUE) , 0(FALSE) , NULL 
TRUE 에 해당하는 값을 반환한다 

## 데이터 종류 
### 1. 데이터형 
![![04.코딩공부/#*Table]]
#### 1-1. 문자형 
* '' 안에 띄어쓰기 주의
* 대소문자 구분 안함
* 끝 띄어쓰기 무시

```SQL
WHERE col_name BINARY 'A' // 정확히 A 만 가져옴 
WHERE col_name LIKE 'A' // = 의미,뒷공백 무시 안함,대소문자 구분안함 
WHERE col_name NOT LIKE 'A'

WHERE col_name LIKE 'A%' // % : 임의의 0개 이상의 문자  약용__, 약용 
WHERE col_name LIKE 'A\%%' // \% :  \ + x특수문자 
WHERE col_name LIKE '%A' // __A, A
WHERE col_name LIKE '%A%' // __A__, A 


WHERE col_name LIKE 'A_' // _ : 임의의 1개의 문자 A_
WHERE col_name LIKE '_A' // _A
WHERE col_name LIKE '_A_' // _A_
```
\n :줄바꿈
\t : 탭
\b : 백스페이스
\r : 복귀 줄바꿈
#### 1-5. 부울형
* 논리형
* IS, IS NOT
* = 을 쓸경우 1,0 으로 판단
### 2. NULL
데이터가 없는 상태 
길이가 0 인 문자열과는 다르다 (name = '';)

```SQL
name IS NULL;
name IS NOT NULL;
1 = NULL // NULL : <=> 이외는 NULL 이 되어버림 
1 != NULL // NULL 
1 <> NULL // NULL 
1 < NULL // NULL 

1 <=> NULL // 0 : FALSE 
NULL <=> NULL // 1 : TRUE 

```

## 논리연산자
```SQL
WHERE price >= 400 AND price < 150; // AND && 
WHERE price >= 400 OR price < 150; // OR || 
WHERE NOT (member_id =1); // NOT ! 
WHERE price >= 100 XOR price < 150; // XOR : 두 조건중 하나만 만족하는 것  
WHERE price >= 100 XOR price < 150 XOR stock >= 100; // 앞 조건 + XOR stock >= 100;  경우 
```
## 편리한 연산자 
```SQL
// BETWEEN
WHERE price BETWEEN 100 AND 150; // price >= 100 AND price <= 150 , 이상-이하
WHERE price NOT BETWEEN 100 AND 150; // not ( price >= 100 AND price <= 150)
// IN : 여러가지 비교 가능 
WHERE product_id IN (1,3,4);
WHERE product_id NOT IN (1,3,4); // NULL 은 안됨 
```
OR , IN 은 많이 쓰면 느려짐 ~ 주의 
## 산술연산자 

## 연산자 우선순위 
![![04.코딩공부/#*Table1]]
() 가 최우선 

# 집계 함수
![![04.코딩공부/#*Table2]]
```SQL
// DISTINCT
SELECT DISTINCT col_name // 중복없이 가져온다
SELECT DISTINCT col_name1, col_name2 // 둘다 중복없이 가져온다 
// COUNT
SELECT COUNT(*) // 행의 수(데이터개수) 세기, 예약어가 아닌 함수, NULL 무시 
```
예약어와 함수의 차이
함수는 input -> output 이 있다 
집계함수는 **SELECT , HAVING , ORDER BY** 에만 적을 수 있다! 
값은 1개씩 반환한다 -> 그러므로 SELECT에 값이 여러개 반환하는것이랑 같이 적을수 없다 
( 상수, 집계함수, DISTINCT, 연산자만 가능! )

# 그룹화 GROUP BY 
그룹마다 집계함수 사용가능 

```SQL
SELECT PROVINCE, AVG(population)
FROM table1
GROUP BY PROVINCE; //PROVINCE 별로 그룹화 하여 population 의 평균을 보여준다 
// PROVINCE : 집계키 

// 여러 집계 사용 
SELECT PROVINCE, AGE, COUNT(*) // SELECT 에 적을수 있음 
FROM table_name 
GROUP BY PROVINCE, AGE // 집계키인것 모두 
// 순서대로 그룹화 
```
그룹함수도 집계함수와 같이 반환값이 1개인 것들만 사용가능하다 
( 상수, 집계함수, 집계키의 컬럼이름=집계키)

### 그룹에 조건주기 
Where -> Having
Where + 조건 : 모든레코드 
Having + 조건 : 그룹 조건 
```SQL
SELECT PROVINCE, COUNT(*)
FROM table_name 
GROUP BY PROVINCE
HAVING COUNT(*) >= 2; // PROVINCE 의 그룹별로 데이터가 2개 이상인것만 
```
Having 에 적을수 있는것 = SELECT 에 적을수 있는것 (그룹화시)
그룹함수도 집계함수와 같이 반환값이 1개인 것들만 사용가능하다 
( 상수, 집계함수, 집계키의 컬럼명)

# 정렬 ORDER BY 
가장 마지막에 적고, 실행순서도 가장 마지막 
```SQL
ORDER BY col_name // col_name : 정렬키 
ORDER BY col_name ASC // col_name 기준 오름차순 정렬 1.2.3.4....
ORDER BY col_name DESC // col_name 기준 내림차순 정렬 4.3.2.1...
ORDER BY col_name1, col_name2, col_name3 
// col_name1 으로 정렬후 그 안에서 col_name2 정렬 후 그 안에서 col_name3 정렬 

SELECT product_name, stock
FROM table_name 
ORDER BY 2 DESC; // 1 = product_name , 2 = stock 
// SELECT * 일경우 순서대로 번호를 생각하면 됨 

SELECT product_name, stock*price
FROM table_name 
ORDER BY stock*price; // SELECT 에 쓰인 값들 가능 
```
정렬키 가 select 에 없어도 된다 
마지막으로 나온 값들에 대해 정렬한다 
NULL 의경우 맨처음,혹은 마지막으로 지맘대로 올수 있으니 값을 매우 작거나 큰 값으로 치환 추천 

```SQL
SELECT * 
FROM table_name
ORDER BY price is NULL ASC, price ASC
// price is NULL 일경우 1(TRUE) 반환 
```
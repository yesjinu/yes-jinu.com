---
title: 데이터베이스 개론과 실습 part 2 (1/2)
date: 2021-07-10 22:07:32
category: dev
thumbnail: { thumbnailSrc }
draft: false
---

하나의 포스팅에서 다루기에 다루는 내용이 방대해 쪼개기로 결정. 3개 분량의 포스팅으로 쪼개는 것이 적절해보인다. 이번 포스팅에서는 기본적인 쿼리 키워드에 대해서 알아본다. 쿼리 키워드를 설명하는 자료는 웹 상에 많으므로, 직접 SQL 문제를 풀어가면서 사고방식을 나열하는 식으로 진행해보려 한다(Learning by doing!). 문제는 [SQLZOO](~https://sqlzoo.net/~)에서 제공하는 퀴즈들을 활용하려 한다. 곧바로 해결방법이 도출되지 않았던 문제와 해결과정에서의 사고과정을 위주로 포스팅한다. 

---
**## SELECT NAMES**

[Link](~https://sqlzoo.net/wiki/SELECT_names~)

서버 개발자로 일하며 가장 많이 수행하게 되는 쿼리는 '특정 문자열'을 찾는 일이다. 특히 트러블슈팅이나 개발 중에 DB에 들어간 값을 '검색'하는 일이 매우 잦다. 

MySQL이 제공하는 string을 편집하는 built-in 함수들과 문자열 패턴 검색 키워드를 연습해보자. 
- `%` : zero or more characters
- `_` : a single character
- `[]` : any single character within the brackets
	- h\[oa\]t 는 'hot'과 'hat'을 모두 찾는다.
- `^` : any character not in the brackets
	- []와 함께 쓰이며, bracket 안에 없는 문자열만 찾는다. 
	- \[^oa\]는 'o', 'a'을 제외한 모든 single chracter를 찾는다.
- `-` : range of characters
	- \[a-c\] 는 a부터 c까지의 알파벳을 찾는다. 

1-13. Find the capital and the name where the capital includes the name of the country.

```sql
SELECT capital, name
FROM world
WHERE capital LIKE concat('%',name,'%');
```

> 처음에는 `WHERE capital LIKE '%name%'`으로 시도했다.
> 이 경우 capital 이름에 'name'이라는 문자열이 포함된 경우를 찾는다.
> ~~당연히 그런 나라는 없다.~~

1-14. Find the capital and the name where the capital is an extension of name of the country.

```sql
SELECT capital, name
FROM world
WHERE capital LIKE concat('%',name,'%') AND capital > name;;
```

> 나는 `capital <> name`으로  찾으려했다. Wild card(%)가 붙어있으니 capital이 더 길겠지 하는 생각으로. 그런데 모범답안에서는 곧바로 string의 대소비교를 하고 있다. SQL이 어떻게 대소비교를 하는지 알아봐야할듯.

1-15. Show the name and the extension where the capital is an extension of name of the country.

```sql
SELECT name, REPLACE(capital, name, '') 
FROM world 
WHERE capital LIKE concat('%', name, '%') AND capital > name;
```

> `REPLACE(word, pattern, substitute)`  word에서 pattern을 찾아 substitute로 대체한다. 아래와 같은 결과를 얻을 수 있다.

| name |  |
| ----------- | ----------- |
| Andorra | la Vella |
| Guatemala | City |
| Kuwait | City |
| Mexico | City |
| Monaco | -Ville |
| Panama | City |

---
[SELECT from WORLD Tutorial - SQLZOO](https://sqlzoo.net/wiki/SELECT_from_WORLD_Tutorial)
2-12. Show the name and the capital where the first letters of each match. Don't include countries where the name and the capital are the same word.
```sql
SELECT name, capital
FROM world
WHERE LEFT(name, 1)=LEFT(capital, 1) AND name <> capital
```
> LEFT라는 built-in 함수로 스트링의 일부분만 받아올 수 있는 점에 주목

**2-13. Find the country that has all the vowels and no spaces in its name.**
```sql
SELECT name
FROM world
WHERE name     LIKE '%a%'
  AND name     LIKE '%e%'
  AND name     LIKE '%i%'
  AND name     LIKE '%o%'
  AND name     LIKE '%u%'
  AND name NOT LIKE '% %';
```
> 문제를 뒤집어서, (a | e | i | o | u)를 모두 포함하지 않는 것을 찾으라고 했다면 헷갈렸을지도. SQL 쿼리문을 배울수록 아주 작은 쿼리 블럭부터 쌓아나간다는 느낌을 받는다.

---

---
## SELECT within SELECT
https://sqlzoo.net/wiki/SELECT_within_SELECT_Tutorial
4-6. Which countries have a GDP greater than every country in Europe? [Give the name only.] (Some countries may have NULL gdp values)

```sql
SELECT name
FROM world
WHERE gdp > ALL(SELECT gdp 
					FROM world
				 	WHERE gdp > 0 AND continent='EUROPE'
					)
```
> ALL이라는 키워드가 있다. 마치 함수형 프로그래밍에서 `map`을 사용하는 것처럼 모든 결과들에 대해 `>` operator를 분배해서 적용시킨다. 결과적으로 MAX 처럼 사용할 수 있는 것!

4-7. Find the largest country (by area) in each continent, show the continent, the name and the area:
```sql
SELECT continent, name, area 
FROM world x
WHERE area >= ALL(SELECT area 
                  FROM world y
                  WHERE x.continent = y.continent 
                  AND y.area > 0);
```
> subquery를 이용한 첫번째 문제. 안쪽 `SELECT`에서 바깥 `SELECT`의 변수를 호출할 수 있다. 바깥쪽 world와 안쪽 world 테이블을 구별하기 aliasing을 사용했다.


---
title: 데이터베이스 개론과 실습 part 2-1.
date: 2021-07-10 22:07:32
category: dev
thumbnail: { thumbnailSrc }
draft: false
---

 하나의 포스팅에서 다루기에 다루는 내용이 방대해 쪼개기로 결정. 3개 분량의 포스팅으로 쪼개는 것이 적절해보인다. 이번 포스팅에서는 기본적인 쿼리 키워드에 대해서 알아본다. 쿼리 키워드를 설명하는 자료는 웹 상에 많으므로, 직접 SQL 문제를 풀어가면서 사고방식을 나열하는 식으로 진행해보려 한다(Learning by doing!). 문제는 [SQLZOO](https://sqlzoo.net/)에서 제공하는 퀴즈들을 활용하려 한다. 곧바로 해결방법이 도출되지 않았던 문제와 해결과정에서의 사고과정을 위주로 포스팅한다. 

---
## SELECT NAMES 
[Link](https://sqlzoo.net/wiki/SELECT_names)

12. Find the country where the capital is the country plus "City".

```sql
SELECT name
  FROM world
 WHERE capital = concat(name, ' City');
```

> 어느 이름(schema, alias)을 어디서부터 사용할 수 있는가 하는 것이 늘 헷갈렸었다. name, capital 등 스키마 이름은 쿼리문 전역에서 사용할 수 있다. `concat`함수를 사용하는 것도 눈여겨볼만 하다.

13. Find the capital and the name where the capital includes the name of the country.

```sql
SELECT capital, name
FROM world
WHERE capital LIKE concat('%',name,'%');
```

> 처음에는 `WHERE capital LIKE '%name%'`으로 시도했다.
> 이 경우 capital 이름에 'name'이라는 문자열이 포함된 경우를 찾는다.
> 당연히 그런 나라는 없다.

14. Find the capital and the name where the capital is an extension of name of the country.

```sql
SELECT capital, name
FROM world
WHERE capital LIKE concat('%',name,'%') AND capital > name;;
```

> 나는 `capital <> name`으로  찾으려했다. Wild card(%)가 붙어있으니 capital이 더 길겠지 하는 생각으로. 그런데 모범답안에서는 곧바로 string의 대수비교(?)를 하고 있다. SQL이 어떻게 대수비교를 하는지 알아봐야할듯.

15. Show the name and the extension where the capital is an extension of name of the country.

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

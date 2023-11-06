# 인덱스

## G를 찾아보세요

{% tabs %}
{% tab title="First Tab" %}
* B
* F
* C
* E
* D
* A
* F
{% endtab %}

{% tab title="Second Tab" %}
* A
* B
* C
* D
* E
* F
* F
{% endtab %}
{% endtabs %}

## 인덱스가 없을경우

| id | name | age |
| -- | ---- | --- |
| 1  | AAA  | 21  |
| 2  | BBB  | 30  |
| 3  | CCC  | 39  |
| 4  | DDD  | 28  |

나이가 가장 적은 사람을 찾고싶다  -> 풀스캔&#x20;



## 인덱스가 있을경우

<table><thead><tr><th width="95">나이</th><th width="152">인덱스 주소</th><th width="59">.</th><th width="131">id</th><th>name</th><th>age</th></tr></thead><tbody><tr><td>21</td><td>1</td><td></td><td>1</td><td>AAA</td><td>21</td></tr><tr><td>28</td><td>4</td><td></td><td>2</td><td>BBB</td><td>30</td></tr><tr><td>30</td><td>2</td><td></td><td>3</td><td>CCC</td><td>39</td></tr><tr><td>39</td><td>3</td><td></td><td>4</td><td>DDD</td><td>28</td></tr></tbody></table>

나이순으로 정렬되어있기 때문에 하나만 조회하면 된다.

인덱스도 `테이블` 이다. :thumbsup:



> 인덱스의 핵심은 탐색 범위를 최소화 하는 것

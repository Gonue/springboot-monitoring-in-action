## Prometheus - Query Language (PromQL_1)

PromQL은 Prometheus 쿼리 언어(Prometheus Query Language)의 약자입니다. Prometheus는 메트릭 데이터를 수집하고, 저장하며, 조회하는 오픈 소스 모니터링 시스템입니다. PromQL은 Prometheus에서 제공하는 강력한 쿼리 언어로, Prometheus 서버에서 메트릭 데이터를 조회하고 분석하는 데 사용됩니다.
PromQL은 간단하고 직관적인 문법을 가지고 있으며, Prometheus에서 메트릭 데이터를 유연하게 탐색하고 분석하는 데 도움이 됩니다.

본 포스트에서는 PromQL을 사용하기 위한 기본 개념들을 정리하고자 합니다.

## Expression Language Data Type

Prometheus는 아래 4가지 데이터 타입으로 구분됩니다.

 - Instant vector - 같은 타임스탬프 상에 있는 시계열 셋으로, 각 시계열마다 단일 샘플을 가지고 있습니다.
 - Range vector - 특정 시간 범위에 있는 시계열 셋으로, 각 시계열마다 시간에 따른 데이터 포인트들을 가지고 있습니다.
 - Scalar - 간단한 부동 소수점 값
 - String - 간단한 문자열 값

## Literals
위 데이터 타입중 Literal 로 표현될 수 있는 타입은 String 과 Scalar 가 있으며 각 정보는 아래와 같습니다.

---

### String literals
문자열은 작은따옴표, 큰따옴표 또는 백틱으로 리터럴로 지정할 수 있으며,
Prometheus는 Go로 만들어진만큼 탈출 문자 규칙도 Go의 규칙을 따릅니다. 

예)
```
"this is a string"
'these are unescaped: \n \\ \t'
`these are not unescaped: \n ' " \t`
```
백틱 내부에서는 이스케이프가 처리 되지 않으며, Go와 달리 Prometheus는 백틱 내부의 줄 바꿈을 버리지 않습니다.

<img width="947" alt="image" src="https://github.com/Gonue/springboot-monitoring-in-action/assets/109960034/207e6277-f0e1-49a2-bf6a-bd7127b37d3c">

---

### Float literals

스칼라의 부동 소수점 값은 다음 형식의 리터럴 정수 또는 부동 소수점 숫자로 작성 할 수 있습니다.
```
[-+]?(
      [0-9]*\.?[0-9]+([eE][-+]?[0-9]+)?
    | 0[xX][0-9a-fA-F]+
    | [nN][aA][nN]
    | [iI][nN][fF]
)
```
실수, 숫자, 지수, 16진수, Inf, -Inf, NaN이 될 수 있습니다.

예시로
```
23
-2.43
3.4e-9
0x8f
-Inf
NaN
```
등을 예시로 들 수 있습니다.
<img width="947" alt="image" src="https://github.com/Gonue/springboot-monitoring-in-action/assets/109960034/b1465e6e-aac0-405c-b7b7-e0f46b96c44f">

<img width="947" alt="image" src="https://github.com/Gonue/springboot-monitoring-in-action/assets/109960034/e8edd6ef-262a-4e4f-99ec-ca45de83e47d">

---

## Time series Selectors

---

### Instant vector

같은 타임스탬프에 있는 Time series 집합이며, 각 Time series 마다 단일 샘플을 가지고 있습니다.
metric 이름으로 쿼리를 실행하면 Instance Vector 를 얻을 수 있으며, 필터링을 하고 싶으면 Instant Vector Selector 를 사용합니다.

예 )
```
system_cpu_usage
```
위 예시는 메트릭명이 system_cpu_usage 인 모든 시계열을 선택합니다.

중괄호( {} ) 안에 쉼표로 구분된 레이블 matcher들을 콤마로 구분해서 넣어주면, 시계열을 구체적으로 필터링 할 수 있습니다.

```
system_cpu_usage{job="spring-actuator-prometheus", group=""}
```

일치하지 않는 레이블 값을 찾거나 정규 표현식과 매칭하는 것도 가능하며, 레이블 매칭에는 다음과 같은 연산자 사용도 가능합니다.

- =: 제공된 문자열과 정확히 동일한 레이블을 선택합니다.
- !=: 제공된 문자열과 같지 않은 레이블을 선택하십시오.
- =~: 제공된 문자열과 정규식 일치하는 레이블을 선택합니다.
- !~: 제공된 문자열과 정규식 일치하지 않는 레이블을 선택합니다.

---

### Range vector
특정 시간 범위에 있는 Time series set 이며,
Instant vector는 각 time-series들이 1개의 Sample만 가지는 것에 반해, Range vector는 지정된 범위의 시간에 속하는 Time series들로 구성되어 있습니다.

Range Vector는 metric이름 뒤에 [시간]을 붙이면 됩니다.

시간은 숫자로 지정되며 바로 뒤에 다음 단위 중 하나가 올 수 있습니다.
```
ms- 밀리초
s- 초
m- 분
h- 시간
d- days - 하루가 항상 24시간이라고 가정
w- 주 - 한 주가 항상 7d라고 가정
y- 년 - 1년이 항상 365d라고 가정
```
주의 사항) 정수만 사용가능 합니다.

예 ) 
<img width="947" alt="image" src="https://github.com/Gonue/springboot-monitoring-in-action/assets/109960034/99a48008-c369-48c1-9f00-0bb83b30ffed">

system_cpu_usage[5m] 는 아래와 같이 분류 됩니다.

- Instant vector selector: system_cpu_usage
- Range vector selector: [ ]
- Time duration: 5m

Selector 로 system_cpu_usage 메트릭을 입력하여 5분간의 Range vector 가 반환됐으며, 

반한된 값으로는 <value> @ <timestamp> 로 구성되어 있습니다.

Range vector 는 반환값이 여러 sample 들이 되므로 보통 사용할때는 단독으로 사용하지 않으며 평균값이나 변동폭(rate)과 같은 aggregation function 을 함께 사용 합니다.

<img width="932" alt="image" src="https://github.com/Gonue/springboot-monitoring-in-action/assets/109960034/1821f5f9-a23b-423d-b05e-e664c205f953">

---

### Offset modifier

Offset modifier 를 사용하면 쿼리에서 개별 순간 및 범위 벡터에 대한 시간 오프셋을 변경 할 수 있습니다.

Prometheus에서 지원하는 Time Duration은 아래와 같다.
<img width="932" alt="image" src="https://github.com/Gonue/springboot-monitoring-in-action/assets/109960034/2ab27780-3b33-4dbf-bd31-1e7dc3b91240">

지난 1분전의 데이터 쿼리를 작성하면 위와 같습니다.

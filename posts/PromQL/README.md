### Prometheus - Query Language (PromQL)

PromQL은 Prometheus 쿼리 언어(Prometheus Query Language)의 약자입니다. Prometheus는 메트릭 데이터를 수집하고, 저장하며, 조회하는 오픈 소스 모니터링 시스템입니다. PromQL은 Prometheus에서 제공하는 강력한 쿼리 언어로, Prometheus 서버에서 메트릭 데이터를 조회하고 분석하는 데 사용됩니다.
PromQL은 간단하고 직관적인 문법을 가지고 있으며, Prometheus에서 메트릭 데이터를 유연하게 탐색하고 분석하는 데 도움이 됩니다.

본 포스트에서는 PromQL을 사용하기 위한 기본 개념들을 정리하고자 합니다.

### Expression Language Data Type

Prometheus는 아래 4가지 데이터 타입으로 구분됩니다.

 - Instant vector - 같은 타임스탬프 상에 있는 시계열 셋으로, 각 시계열마다 단일 샘플을 가지고 있습니다.
 - Range vector - 특정 시간 범위에 있는 시계열 셋으로, 각 시계열마다 시간에 따른 데이터 포인트들을 가지고 있습니다.
 - Scalar - 간단한 부동 소수점 값
 - String - 간단한 문자열 값

### Literals
위 데이터 타입중 Literal 로 표현될 수 있는 타입은 String 과 Scalar 가 있으며 각 정보는 아래와 같습니다.

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

 



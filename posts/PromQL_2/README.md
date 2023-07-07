## Prometheus - Query Language (PromQL_2)

해당 목차는 [프로메테우스 공식 레퍼런스](https://prometheus.io/docs/prometheus/latest/querying/operators/)를 참조하여 요약하였습니다.

## Operators(연산자)

### Binary operators

Prometheus 의 쿼리언어는 기본적인 논리 연산자와 산술 연산자를 지원하며, 두 개 instant 벡터를 연산할 땐 매칭 동작 수정이 가능합니다.

#### Arithmetic binary operators
```
+ (더하기)
- (빼기)
* (곱하기)
/ (나누기)
% (나머지)
^ (거듭제곱/지수)
```


#### Comparison binary operators

```
Comparison binary operators
== (동일한)
!= (같지 않음)
> (보다 큰)
< (보다 작은)
>= (크거나 같음)
<= (작거나 같)
```

#### Logical/set binary operators

```
Logical/set binary operators
and (교집합)
or (합집합)
unless (여집합)
```

#### Aggregation Operators
```
sum (차원들의 합계를 계산)
min (차원들에서 최소값을 선택)
max (차원들에서 최대값 선택)
avg (차원들의 평균 계산)
stddev (모든 값을 1로 가지고 있는 벡터 생성)
stdvar (차원들의 모분산 계산)
count (벡터에 있는 요소의 갯수 카운트)
count_values (같은 값을 가지고 있는 요소들의 갯수 카운트)
bottomk (샘플 값이 가장 작은 요소 k개)
topk (샘플 값이 가장 큰 요소 k개)
quantile (차원들의 φ-quantile (0 ≤ φ ≤ 1) 을 계산)
```

---

### Vector Matching
PromQL 의 가장 강력한 기능입니다.

Instant vector 들을 라벨을 통해 매칭하여 값들을 연산하거나 라벨을 결합 할 수 있습니다.

### One - to one matching
벡터가 1:1로 정확히 일치하는 경우 사용하며(라벨 매칭으로 두 개 이상의 벡터들이 1:1로 매칭되는 경우), ignoring 키워드를 통해 특정 라벨을 무시하거나, on 키워드로 제공되는 라벨리스트를 특정 라벨들로 구성되도록 할 수 있다.
```
<vector expr> <bin-op> ignoring(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) <vector expr>
```

##### Example Input
```
method_code:http_errors:rate5m{method="get", code="500"}  24
method_code:http_errors:rate5m{method="get", code="404"}  30
method_code:http_errors:rate5m{method="put", code="501"}  3
method_code:http_errors:rate5m{method="post", code="500"} 6
method_code:http_errors:rate5m{method="post", code="404"} 21

method:http_requests:rate5m{method="get"}  600
method:http_requests:rate5m{method="del"}  34
method:http_requests:rate5m{method="post"} 120
```

##### Example query
```
method_code:http_errors:rate5m{code="500"} / ignoring(code) method:http_requests:rate5m
```

##### Example output
```
{method="get"}  0.04            //  24 / 600
{method="post"} 0.05            //   6 / 120
```

---

### One-to-many / Many-to-one matching
one 쪽의 벡터가 many 쪽의 벡터와 일치 할 수 있는 경우에 사용되며, 해당 케이스에서 group_left modifier 나 group_right modifier 를 통해 더 높은 cardinality 를 갖는 쪽을 지정해 줘야 합니다.

```
<vector expr> <bin-op> ignoring(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> ignoring(<label list>) group_right(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_right(<label list>) <vector expr>
```

### Example Query
```
method_code:http_errors:rate5m / ignoring(code) group_left method:http_requests:rate5m
```

### Example Output
```
{method="get", code="500"}  0.04            //  24 / 600
{method="get", code="404"}  0.05            //  30 / 600
{method="post", code="500"} 0.05            //   6 / 120
{method="post", code="404"} 0.175           //  21 / 120
```

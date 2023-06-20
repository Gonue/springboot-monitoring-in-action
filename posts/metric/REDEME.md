### Spring Actuator - 기본 메트릭 정보

Spring Actuator는 Spring Boot 애플리케이션의 운영 환경에서 모니터링과 관리를 위한 기능을 제공하는 라이브러리입니다. 이 라이브러리는 애플리케이션의 다양한 측면을 모니터링할 수 있는 엔드포인트(endpoint)를 제공하며, 이 중 일부는 메트릭(metric)을 수집할 수 있습니다. 메트릭은 애플리케이션의 상태와 동작에 대한 정보를 수집하고, 이를 통해 애플리케이션의 성능, 가용성, 문제점 등을 분석할 수 있습니다.


### HTTP 엔드포인트 관련 메트릭:

- http.server.requests: 이 메트릭은 HTTP 요청과 관련된 다양한 지표를 수집합니다. 몇 가지 예시는 다음과 같습니다:

  - http.server.requests.count: 요청의 총 수를 카운트합니다.
  - http.server.requests.method.GET: GET 메서드에 대한 요청 수를 카운트합니다.
  - http.server.requests.method.POST: POST 메서드에 대한 요청 수를 카운트합니다.
  - http.server.requests.method.{METHOD_NAME}: 특정 HTTP 메서드에 대한 요청 수를 카운트합니다.
  - http.server.error: 이 메트릭은 서버에서 발생한 HTTP 오류 상태 코드에 대한 정보를 수집합니다. 몇 가지 예시는 다음과 같습니다:
  - http.server.error.count: 발생한 오류 상태 코드의 총 수를 카운트합니다.
  - http.server.error.{STATUS_CODE}: 특정 HTTP 오류 상태 코드에 대한 수를 카운트합니다.


### JVM 관련 메트릭:

- jvm.memory: 이 메트릭은 JVM의 힙 메모리 사용량과 GC에 대한 정보를 수집합니다. 일부 예시는 다음과 같습니다:
  - jvm.memory.max: JVM의 힙 메모리 최대 크기를 나타냅니다.
  - jvm.memory.used: 현재 사용 중인 JVM 힙 메모리 양을 나타냅니다.
  - jvm.memory.committed: JVM에 의해 현재 커밋된 힙 메모리 양을 나타냅니다.
  - jvm.gc: GC의 실행 횟수와 지연 시간 등에 대한 정보를 수집합니다.

- jvm.threads: 이 메트릭은 JVM 내의 쓰레드 관련 정보를 수집합니다. 예시로는 다음과 같습니다:

  - jvm.threads.live: 현재 실행 중인 쓰레드의 수를 나타냅니다.
  - jvm.threads.peak: 애플리케이션 실행 도중 가장 많이 동시에 실행된 쓰레드의 수를 나타냅니다.


### 데이터베이스 관련 메트릭:
- datasource.*: 이 메트릭은 데이터베이스 연결 풀에 대한 정보를 수집합니다. 데이터베이스 종류 및 구성에 따라 다양한 메트릭이 제공될 수 있습니다. 예를 들면:
  - datasource.primary.active: 현재 활성 상태인 데이터베이스 연결의 수를 나타냅니다.
  - datasource.primary.max: 데이터베이스 연결 풀의 최대 크기를 나타냅니다.
  - datasource.primary.usage: 데이터베이스 연결 풀의 사용률을 나타냅니다.

### 시스템 관련 메트릭:

- system.cpu: 이 메트릭은 CPU 사용량과 로드에 관련된 정보를 수집합니다. 몇 가지 예시는 다음과 같습니다:

  - system.cpu.count: 시스템의 총 CPU 코어 수를 나타냅니다.
  - system.cpu.usage: 시스템의 CPU 사용률을 나타냅니다.
  - system.memory: 이 메트릭은 시스템 메모리 사용량과 가용성에 관련된 정보를 수집합니다. 예시로는 다음과 같습니다:
  - system.memory.max: 시스템의 총 메모리 용량을 나타냅니다.
  - system.memory.used: 현재 사용 중인 시스템 메모리 양을 나타냅니다.

### 기타 메트릭:

- process.*: 이 메트릭은 애플리케이션 프로세스와 관련된 정보를 수집합니다. 예시로는 다음과 같습니다:

  - process.start.time: 애플리케이션 프로세스의 시작 시간을 나타냅니다.
  - process.cpu.usage: 애플리케이션 프로세스의 CPU 사용률을 나타냅니다.
  - logback.* or log4j2.*: 이 메트릭은 로그 관련 메트릭을 수집합니다. 로그 레벨, 로그 메시지 수 등과 같은 정보를 포함할 수 있습니다.

Spring Actuator를 사용하여 수집할 수 있는 메트릭은 개발자가 원하는 경우 확장 가능합니다. 자체적으로 사용자 정의 메트릭을 수집하고 노출할 수도 있습니다. 이를 위해 Actuator API를 사용하여 애플리케이션에 맞는 메트릭을 추가로 구현할 수 있습니다.


Spring Actuator 공식문서

>  https://docs.spring.io/spring-boot/docs/current/actuator-api/htmlsingle/#prometheus
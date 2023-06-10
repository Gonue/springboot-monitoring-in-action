# 환경 구축

## 순서
1. Spring Boot 설정
2. Prometheus 설치
3. Prometheus target 설정
4. Grafana 설치
5. Grafana 대시보드 구성

## 1. Spring Boot 설정 
### 1-1 의존성

Spring Boot Actuator는 스프링부트의 서브 프로젝트이며 스프링부트 애플리케이션에서 SpringBoot Actuator를 활성화하면 애플리케이션을 모니터링하고 관리 할 수 있는 엔드포인트에 접속이 가능해 진다.

```shell
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    runtimeOnly 'io.micrometer:micrometer-registry-prometheus'
}
```

Gradle 기준으로 의존성을 위와 같이 추가하면, Actuator가 활성화 된다. 그리고 Prometheus에서 사용되는 메트릭을 수집할 때 micrometer-registry-prometheus 라는 의존성이 요구 되므로 해당 의존성도 추가 해준다.

### 1-2 yml설정
```yaml
management:
  endpoints:
    web:
      exposure:
        include: 
          - prometheus
```

위 yml설정은 Spring Boot Actuator에 대한 설정이며 여러가지 Actuator에 대한 설정이 있지만 이 포스팅엔 prometheus에서 사용되는 메트릭에 대한 엔드포인트의 노출만 필요하므로 include에 prometheus만 기입하여 설정한다.

이러한 엔드포인트 중 일부는 민감한 정보를 노출할 수 있으므로 보안설정에 주의하며 Spring Actuator에서 제공하는 엔드포인트 전체는 아래 공식문서에서 확인 가능하다.
> https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints

애플리케이션을 실행후 http://localhost:8080/actuator 로 들어가면 prometheus 엔드포인트가 노출된것을 확인 할 수 있다.

<img width="634" alt="image" src="https://github.com/Gonue/haversine-formula-ex/assets/109960034/6b586e61-623e-46a2-887d-351973c6eec1">

그리고 노출된 prometheus 엔드포인트로 접속시 아래와 같은 메트릭 데이터가 확인 가능하며 prometheus는 이 엔드포인트에 접속하여 주기적으로 메트릭 정보를 수집한다.

<img width="1486" alt="image" src="https://github.com/Gonue/haversine-formula-ex/assets/109960034/d36e6c59-3a25-4208-bbca-239dc41577e7">

## 2. Prometheus 설치

Prometheus는 오픈소스 모니터링 시스템으로, 메트릭 데이터를 수집하고 저장하여 모니터링하고 경고를 설정할 수 있게 도와준다. 
Pull 모델을 사용하여 주기적으로 메트릭 데이터를 가져오며, 클라이언트에 에이전트 설치가 필요하지 않다. 간단하게 말하면, Prometheus는 숫자 데이터를 수집하고 분석하여 시스템 상태를 모니터링하는 도구이다.

Prometheus는 클라우드 환경에서 많이 사용되며, Kubernetes와 같은 컨테이너 오케스트레이션 시스템과의 통합이 강조된다.
그래서 이 포스팅에선 도커로 간단히 Prometheus 설치를 진행합니다.

> https://prometheus.io/docs/prometheus/latest/installation/#using-docker

> https://hub.docker.com/r/prom/prometheus/

위는 prometheus의 공식문서와 DockerHub 주소이며 위 부분을 참고하여 설치하였습니다.

## 3. Prometheus target 설정

원하는 경로에 prometheus.yml 파일을 생성하고 아래와 같이 내용을 작성합니다.
```yaml
global:
  scrape_interval: 10s
  evaluation_interval: 10s
scrape_configs:
  - job_name: 'spring-actuator-prometheus'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8080']
```

- global : 전역에 적용되는 룰
- scrape_interval : 타겟으로부터의 메트릭을 스크랩(수집) 주기
- evaluation_interval : 룰을 평가하는 간격설정
    - 룰은 Prometheus가 수집한 메트릭 데이터를 기반으로 경고(alerting)를 생성하거나, 데이터를 가공처리하는 등 의 작업을 수행
- scrape_configs : Prometheus가 메트릭을 수집할 타켓 정의 부분
- job_name : 메트릭 수집대상 식별 이름
  - ex)여러 개의 서버에서 실행 중인 웹 애플리케이션의 메트릭을 수집한다고 가정했을때 각 서버의 메트릭을 구분하기 위해 Prometheus가 사용하는 각 서버의 고유 식별자 이름
- metrics_path : 메트릭을 수집하기 위해 타겟으로부터 메트릭을 가져올 경로를 지정하는 설정(actuator 때 설정한 prometheus 노출 엔드포인트)
- static_configs : 타겟 목록을 정의하는 부분
- tragets : 메트릭 수집대상(IP 주소 또는 도메인과 포트 번호로 구성)
  - Docker 컨테이너 내에서 호스트 시스템을 가리키는 특별한 도메인 이름. 즉 Docker 컨테이너 내부에서 호스트 시스템의 IP주소가 있는 8080포트에 대한 연결을 의미
  - 개발용으로만 사용되야 하며, Docker Destop 외부의 환경에서는 동작 x

위에 사용된 항목들은 매우 기초적인 사항이며, service discovery, Alerting Rule 등 상세 파라미터들은 추후 다른 곳에서 포스팅 예정이며 아래 공식문서에서 확인가능.
> https://prometheus.io/docs/prometheus/latest/configuration/configuration/

> https://prometheus.io/docs/guides/file-sd/#use-file-based-service-discovery-to-discover-scrape-targets

docker 컨테이너가 정상적으로 생성 되었다면 http://localhost:9090 포트로 들어가면 Prometheus 확인 가능.
<img width="989" alt="image" src="https://github.com/Gonue/haversine-formula-ex/assets/109960034/8fcadb49-b454-4bb4-a0cf-1834a907eebf">
http://localhost:9090/config 에서 전체설정 확인이 가능하며 
<img width="989" alt="image" src="https://github.com/Gonue/haversine-formula-ex/assets/109960034/d03d8813-aae0-4b56-802b-9db04dac82b7">
http://localhost:9090/targets 으로 타겟과의 상태 확인이 가능하며 정상적이면 UP 이라 표시됩니다.
<img width="1780" alt="image" src="https://github.com/Gonue/haversine-formula-ex/assets/109960034/4cfe7f94-0625-479d-a98b-e94cb7c58d66">

그리고 메인으로 돌아가 jvm_memory_used_bytes와 같이 쿼리를 날리면 연결된 애플리케이션의 데이터를 시각화 또는 테이블로 볼 수 있습니다.
<img width="1885" alt="image" src="https://github.com/Gonue/haversine-formula-ex/assets/109960034/b7845b40-ce6c-4f9b-9372-6448958580ba">

Prometheus의 기본적인 설정은 끝으로 이제 주기적으로 수집된 메트릭 데이터를 Grafana를 통해 표시해 보겠습니다.

## 4. Grafana 설치

Grafana는 오픈소스 데이터 시각화 및 메트릭 분석 도구로서 대시보드를 생성하고 다양한 데이터 소스에서 메트릭 데이터를 시각적으로 표현하고 분석할 수 있게 해줍니다. 주로 시스템 및 애플리케이션 모니터링에 사용되며, 다양한 데이터 소스와의 통합이 가능하며 그중 Prometheus도 포함됩니다.

> https://grafana.com/docs/grafana/latest/setup-grafana/installation/docker/

똑같이 위 공식문서를 참조하여 docker로 진행하였습니다.
```shell
docker run -d --name=grafana -p 3000:3000 grafana/grafana
```
![image](https://github.com/Gonue/haversine-formula-ex/assets/109960034/39cb0c44-50f2-4bb0-942c-82f6625dd450)
진행이 잘 되었다면 http://localhost:3000 포트로 접속하면 Grapana 로그인 화면이 나오며 초기 Id/password는 admin/admin 입니다.

## 5. Grafana 대시보드 구성
먼저 이전에 설치한 Prometheus를 Data source로 설정합니다.

좌측 상단 햄버거 버튼 -> Administration -> Data sources -> Add new data source로 이동하여 Prometheus를 선택합니다.
> http://localhost:3000/datasources/new

<img width="1206" alt="image" src="https://github.com/Gonue/haversine-formula-ex/assets/109960034/0e5b40f5-617e-4d23-aa9a-b87d938d71dd">

URL에 Prometheus Url을 입력후 하단의 Save % test를 클릭하여 저장후 Data source is working 이라는 문구가 나오면 성공 입니다.

이후 대시보드 -> new dashboard 선택후 보고싶은 데이터 쿼리를 입력후 쿼리를 날려가며 원하는 데이터가 조회되면 해당 대시보드를 사용하시면 됩니다.


## + 추가

대시보드 같은 경우 직접 구성하여 사용하여도 무방하지만 잘 설정된 프리셋을 사용하면 더 효과적으로 사용하실수 있습니다.
예시로 JVM (Micrometer) 라는 대시보드를 사용해 보겠습니다.

Dashboards 페이지로 이동후 New -> Import를 누르면 아래와 같은 페이지가 나오는데
<img width="680" alt="image" src="https://github.com/Gonue/haversine-formula-ex/assets/109960034/242746cb-6046-4e91-9789-0f58d7e5845d">

Import via grafana.com form에  - 예시로 JVM(Micrometer)라는 대시보드 사용
> ex) https://grafana.com/grafana/dashboards/4701

입력후 Load -> Prometheus에서 추가한 Prometheus Data Source를 선택하여 Import 해줍니다.

그럼 방금 추가한 JVM(Micrometer) 대시보드 확인이 가능합니다.
<img width="1891" alt="image" src="https://github.com/Gonue/haversine-formula-ex/assets/109960034/17f90b57-1987-49bc-90fb-0c18466a5bbf">

이후에도 다양한 대시보드가 있으며 Import받기를 원하시면 아래 사이트를 참고 하시면 될 것 같습니다.
> https://grafana.com/grafana/dashboards/

> https://grafana.com/tutorials/

<img width="1891" alt="image" src="https://github.com/Gonue/haversine-formula-ex/assets/109960034/4b267446-a303-428d-b205-efdaa786b7ab">

---
설정은 대부분 간단하지만 앞으로 Grapa를 잘 활용하여 애플리케이션에 어떤부분에서 부하를 받고 어떤 상황인지에 대한 적절한 매트릭정보를 불러와 시각화 하는것이 중요한 부분 같습니다.

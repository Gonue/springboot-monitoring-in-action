### Email을 통한 Alert 기능

Metric 임계치를 설정하여 임계치를 넘으면 여러가지 Alert 기능을 통해 실시간으로 서버의 상태를 모니터링 가능합니다.

## 순서
1. Grafana 서버 설정
2. Grafana 콘솔 설정

## Grafana 서버 설정

전 포스팅과 설정은 동일합니다.

docker 상태 확인 후 Grafana CONTAINER ID 나 NAME 을 확인 합니다.
```shell
docker ps
```
docker (Grafana)접속
```shell
docekr exec -u root -it [CONTAINER ID] /bin/bash
```
이후 에디터로 Grafana 설정파일을 열어 줍니다.(defaults.ini) 
<img width="547" alt="image" src="https://github.com/Gonue/springboot-monitoring-in-action/assets/109960034/03736fcb-38f4-436b-a72e-f3d45825e61e">

파일에서 smtp 섹션을 찾아 아래와 같이 수정합니다.

```shell
[smtp]
enabled = true
host = smtp.gmail.com:587
user = xxxx@gmail.com
# If the password contains # or ; you have to wrap it with triple quotes. Ex """#password;"""
password = xxxx
cert_file =
key_file =
skip_verify = false
from_address = admin@grafana.localhost
from_name = Grafana
ehlo_identity =
startTLS_policy =
```
- enabled: SMTP 서비스를 활성화할지 여부를 지정합니다. true로 설정하면 활성화됩니다.
- host: SMTP 서버의 호스트 및 포트를 지정합니다. smtp.gmail.com:587은 Gmail SMTP 서버를 사용하는 예시입니다.
- user: SMTP 서버에 인증하기 위한 사용자 계정을 지정합니다. 여기서는 활성화할 계정을 입렵합니다.
- password: SMTP 서버에 인증하기 위한 비밀번호를 지정합니다. 
  - #이나 ;와 같은 특수 문자가 포함된 경우, 비밀번호를 삼중 따옴표(""")로 감싸야 합니다.
- cert_file: 필요한 경우, SSL/TLS 인증서 파일의 경로를 지정할 수 있습니다.
- key_file: 필요한 경우, SSL/TLS 키 파일의 경로를 지정할 수 있습니다.
- skip_verify: SSL/TLS 인증서의 유효성 검사를 건너뛸지 여부를 지정합니다. false로 설정하면 인증서 검사가 수행됩니다.
- from_address: 이메일 발신자의 주소를 지정합니다.
- from_name: 이메일 발신자의 이름을 지정합니다.
- ehlo_identity: 필요한 경우 EHLO 식별자를 지정할 수 있습니다.
- startTLS_policy: 필요한 경우 STARTTLS(Transport Layer Security) 정책을 지정할 수 있습니다.

아래는 Grafana 설정 공식 문서 입니다.
> https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/

위 와 같이 설정이 끝났다면 컨테이너를 다시 시작해 줍니다.
```shell
docker restart [CONTAINER ID]
```

## Grafana 콘솔 설정

Grafana로 돌아와 좌측 상단 햄버거 메뉴 -> Alerting -> Contact points에서 Contact points를 추가 합니다.

<img width="1495" alt="image" src="https://github.com/Gonue/springboot-monitoring-in-action/assets/109960034/07c96a7e-1b9b-42b4-b46e-3845be6b52b1">

Addresses 부분에 알림을 받을 이메일을 입력후 Test를 누르시면 아래와 같이 성공 메시지가 나타납니다.

<img width="379" alt="image" src="https://github.com/Gonue/springboot-monitoring-in-action/assets/109960034/e8a0b6cd-ecd9-4f65-9e84-cf9ee95e80c4">

이후 이메일을 확인해보시면 아래와 같은 내용을 확인할 수 있습니다.

<img width="1081" alt="image" src="https://github.com/Gonue/springboot-monitoring-in-action/assets/109960034/fda38339-f5ef-4274-8e28-d27ec400083d">

그리고 Contact points 저장 합니다.

## Dashboard 설정

저는 프리셋 중에 SpringBoot APM Dashboard 에서 진행하며 CPU 사용량에 대한 알람을 추가해 보겠습니다.

<img width="1911" alt="image" src="https://github.com/Gonue/springboot-monitoring-in-action/assets/109960034/9615ad71-aa4b-47b8-93b9-84f2de9e21cf">

패널 중에 CPU Usage 에서 설정을 누른후 Edit을 눌러줍니다.
<img width="1019" alt="image" src="https://github.com/Gonue/springboot-monitoring-in-action/assets/109960034/bb0ea72b-0144-4a55-ad6e-99a55c185f34">

이후 Query 섹션에서 아래 부분을 수정해 줍니다.
<img width="1487" alt="image" src="https://github.com/Gonue/springboot-monitoring-in-action/assets/109960034/1fb23cac-b72e-4489-957a-b6fd680289a3">

경고 설정을 위해선 템플릿 변수를 사용할 수 없습니다.(템플릿 변수가 prometheus 쿼리에서 사용될때 Grafana에선 경고가 지원되지 않습니다.)
그래서 해당 $표시를 제거 하여 수정합니다.

<img width="1487" alt="image" src="https://github.com/Gonue/springboot-monitoring-in-action/assets/109960034/29db4bbd-6659-4a09-b122-ef2c59293d0c">
system_cpu_usage{instance="instance", application="application"} 이런식으로 수정하여도 되지만 저는 사용중인 인스턴스나 애플리케이션이 하나 이므로 system_cpu_usage 빼곤 제거하여 수정하겠습니다.


패널에 있는 데이터 외 알림을 생성 하고 싶다면 우측 상단 햄버거 버튼 에서 Alerting -> Alert rules -> Create alert rule 에서 알림을 생성할 수도 있습니다.

<img width="335" alt="image" src="https://github.com/Gonue/code-traveler-mirror/assets/109960034/7cfc14cd-55ea-4e88-bec5-bd7358ad7d40">

이후 5가지 섹션으로 나뉘어져 있는데 하나씩 살펴 보겠습니다.

<img width="1165" alt="image" src="https://github.com/Gonue/code-traveler-mirror/assets/109960034/842e8818-4b65-4a2f-81c7-271a4aabeef4">

1. Set an alert rule name (알림 규칙 이름): 알림 규칙에 대한 이름을 지정합니다. 이는 해당 알림 규칙을 식별하는 데 사용됩니다. 예시로 패널에서 선택했던 "CPU Usage" 로 설정되었습니다.

<img width="1496" alt="image" src="https://github.com/Gonue/code-traveler-mirror/assets/109960034/6674958a-93b5-4f96-a7c5-9ede7558f6a8">

2. Set a query and alert condition : 쿼리와 알림 조건을 설정하는 단계입니다. 이 단계에서는 두 개의 쿼리(A와 B)를 설정하고, 각 쿼리에 대한 알림 조건을 정의합니다.
간단히 작성후 살펴 보면 아래와 같습니다.

A, B

- Local-PrometheusLocal-Prometheus: 이 부분은 쿼리를 실행할 데이터 소스를 나타냅니다. 여기서는 로컬 Prometheus 서버를 사용하고 있습니다.
- now-6h to now: 이 시간 범위는 최근 6시간 동안의 데이터를 쿼리하도록 설정한 것입니다.
- Max data points: 쿼리 결과로 반환될 수 있는 데이터 포인트의 최대 개수를 설정합니다. 여기서는 43200개를 설정하였습니다.
- Metrics browser: 쿼리할 메트릭(데이터)를 선택하는 곳입니다.
- system_cpu_usage (A의 Metrics browser)와 process_cpu_usage{instance=~".*", application=""} (B의 Metrics browser): 시스템 전체의 CPU 사용률과 특정 프로세스의 CPU 사용률을 각각 쿼리합니다.
- Options: 여기서는 각 시리즈의 범례 이름(legend)과 결과 포맷, 쿼리 간격(step) 등을 설정할 수 있습니다.

C

- Reduce: 여러 개의 데이터 포인트를 하나로 줄여서 표현하는 방법을 설정합니다.
- Function: 여기서는 'Last'를 선택하였으므로, 가장 최근의 데이터 포인트를 사용하게 됩니다.
- Mode: 'Strict' 모드는 모든 데이터 포인트를 포함하여 계산하라는 의미입니다.

D

- Threshold: 알림을 발생시키는 임계값을 설정하는 부분입니다.
- IS ABOVE 0: 이 알림 조건은 '값이 0보다 큰 경우'를 나타냅니다. 따라서 시스템 CPU 사용률이나 프로세스 CPU 사용률이 0보다 큰 경우 알림이 발생하도록 설정한 것입니다.
- 'Add query'와 'Add expression'은 추가적인 쿼리나 표현식을 입력하는 부분이며, 'Preview'는 현재까지의 설정에 따른 알림 상태를 미리 확인하는 기능입니다.

<img width="750" alt="image" src="https://github.com/Gonue/code-traveler-mirror/assets/109960034/50c3e3cb-856d-43c5-a65f-1be5c6de4f8e">

3. Alert evaluation behavior : 알림 평가의 행동과 관련된 설정을 하는 섹션입니다.

- Folder: 알림 규칙이 저장될 폴더를 선택하는 부분입니다. 여기서는 'test' 폴더를 선택하였습니다.
- Evaluation group (interval): 동일한 시간 간격에 대해 동일한 그룹 내의 모든 규칙을 평가하는 그룹을 선택하는 부분입니다. 여기서는 '20m' 그룹을 선택하였고, 이 그룹 내의 알림 규칙들은 모두 20초마다 평가됩니다.
- Edit evaluation group: 그룹 평가 시간을 편집하는 부분입니다. 여기서는 10분으로 설정하였습니다.

- Pause evaluation : 이 옵션을 선택하면, 알림 규칙의 평가를 일시 중지할 수 있습니다.

- Configure no data and error handling

   - Alert state if no data or all values are null: 데이터가 없거나 모든 값이 null인 경우의 알림 상태를 설정합니다. 여기서는 'Alerting' 상태로 설정하였으므로, 해당 조건에서 알림이 발생하게 됩니다.
   - Alert state if execution error or timeout: 실행 오류나 시간 초과가 발생한 경우의 알림 상태를 설정하는 부분입니다. 여기서는 'Error' 상태로 설정하였으므로, 해당 조건에서 알림이 발생하게 됩니다.


4. Add details for your alert rule : 이 부분은 알림 규칙의 세부사항을 추가하는 곳입니다.

- Summary and annotations: 여기서는 알림 규칙에 대한 요약 정보와 주석을 작성할 수 있습니다. 알림을 더욱 효과적으로 관리하기 위해 이 정보를 사용할 수 있습니다.

- Dashboard UID: 이 곳에는 대시보드의 고유 식별자가 들어갑니다. 여기서는 'X034JGT7Gz'가 대시보드의 UID입니다.

- Panel ID: 이 부분은 특정 대시보드 패널의 ID를 지정합니다. 여기서는 '95'번 패널이 지정되었습니다.

- Choose: 여기에서는 알림의 내용을 직접 작성하거나, 템플릿을 사용하여 내용을 자동으로 생성할 수 있습니다.

- Add annotation: 이 옵션을 사용하면, 알림에 추가 정보를 제공하는 주석을 추가할 수 있습니다. 주석은 알림을 더욱 명확하게 이해하는 데 도움이 됩니다.

- Set dashboard and panel: 이 부분에서는 알림이 참조할 대시보드와 패널을 선택합니다. 이 설정은 알림이 발생했을 때, 어떤 대시보드와 패널에서 문제가 발생했는지를 식별하는 데 도움이 됩니다.


5. Notifications : 이 부분은 Grafana에서 알림에 대한 알림 설정을 조정하는 곳입니다.

- Grafana handles the notifications for alerts by assigning labels to alerts: Grafana는 알림에 레이블을 할당함으로써 알림의 알림을 처리합니다. 이 레이블들은 알림을 연락처 포인트에 연결하고 일치하는 레이블이 있는 알림 인스턴스를 무시하게 합니다.

- Root route – default for all alerts: Root route은 모든 알림에 대한 기본 경로입니다. 사용자 정의 레이블 없이 알림은 root route를 통해 전송됩니다.

- Custom Labels
  - Labels: 알림에 할당할 사용자 정의 레이블을 설정하는 부분입니다. 레이블은 key와 value의 쌍으로 이루어집니다.
  - Choose key와 Choose value: 레이블의 키와 값을 선택하는 부분입니다. 이 레이블은 알림이 전송되는 대상과 알림의 중요도 등을 결정하는 데 사용될 수 있습니다.
  - 
이 설정을 통해, 알림이 어떤 연락처 포인트로 전송되는지, 특정 알림을 어떻게 처리할지 등을 세밀하게 조절할 수 있습니다.

이와 같이 설정후 Alert rules 페이지 이동후 확인하면 설정이 완료된 것이 확인 가능 합니다.

따라서, 설정에 따르면 알림 규칙은 매 20초마다 평가되고, 이 과정에서 시스템의 전체 CPU 사용률 중 마지막 데이터 포인트가 0 이상인 경우 알림이 생성되며, 이 알림은 root route(기본 이메일)를 통해 전송됩니다.

그리고 설정한 메일을 확인해보면 아래와 같은 메일을 확인 할 수 있습니다.

<img width="750" alt="image" src="https://github.com/Gonue/code-traveler-mirror/assets/109960034/a04cfda4-0c5c-44b4-bbeb-5e3237ee057d">

위에 진행한 사항은 아주 간단히 테스트만 해본 사항이며 이 후엔 알림을 이메일 외에도 웹훅을 이용하여 slack 이나 discord 등 다양한 서비스에 알림 연동하고, 아래와 같은 사용을 생각해 볼 필요가 있습니다.

- 알림 분류: 모든 알림이 동일하게 중요하지는 않습니다. 알림을 중요도, 관련 시스템, 알림 유형 등에 따라 분류하면 우선 순위를 결정하고 더 중요한 이슈에 먼저 대응하는 데 도움이 됩니다.

- 알림 라우팅: 알림을 적절한 팀이나 개인에게 라우팅하도록 설정하세요. 이는 불필요한 정보의 과부하를 줄이고 관련 있는 사람들이 적시에 알림을 받아 처리할 수 있도록 합니다.

- 알림 조건 최적화: 너무 많은 알림은 "알림 피로"를 유발하고 중요한 알림이 묻히게 만들 수 있습니다. 반대로, 너무 적은 알림은 중요한 이슈를 놓칠 수 있습니다. 알림 조건을 지속적으로 최적화하여 이러한 문제를 방지하세요.

- 알림 문서화: 알림이 무엇을 의미하는지, 어떻게 대응해야 하는지 등을 문서화하고 이를 모든 관련 팀원과 공유하세요. 이는 빠르고 일관된 대응을 가능하게 합니다.

- 알림 리뷰: 주기적으로 알림 성능을 리뷰하고 필요한 경우 알림 설정을 조정하세요. 이는 시스템의 변경, 새로운 이슈 패턴 등에 적응할 수 있게 합니다.

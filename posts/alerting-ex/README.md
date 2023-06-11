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

이후 Alert 섹션으로 이동후 Create alert rule from this panel을 클릭해 줍니다. 그럼 rule 설정 페이지로 이동 되며 아래와 같이 작성합니다.





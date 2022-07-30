# Resource Monitor Plugin

## 사용법

다음과 같이 `config.ini`  또는 명령줄에서 사용할 수 있습니다.&#x20;

```
# config.ini
plugin = eosio::resource_monitor_plugin
[options]

# command-line
nodeos ... --plugin eosio::resource_monitor_plugin [options]
```

## Resource Monitor Plugin 설정 옵션

다음은 nodeos 명령줄 또는 `config.ini` 파일에 설정할 수 있는 옵션들 입니다.

```
eosio::resource_monitor_plugin 설정 옵션
  --resource-monitor-interval-seconds arg (=2)
    리소스 사용 체크 시간 간격(초 단위). 1~300 사이.
  --resource-monitor-space-threshold arg (=90)
    사용된 공간 대 전체 공간의 백분율로 나타낸 임계값. 
    사용된 공간이 (임계값 - 5%) 이상일 경우 경고 발생.
    resource-monitor-not-shutdown-on-threshold-exceeded를 사용하도록 설정하지 않은 경우 
    사용된 공간이 임계값을 초과하면 정상 종료됨.값은 6~ 99 사이.
  --resource-monitor-not-shutdown-on-threshold-exceeded 
    임계값이 넘어도 nodeos 를 종료하지 않음.
  --resource-monitor-warning-interval arg (=30)
    임계값에 다다랐을 때 발생하는 경고들 사이에 수행하는 리소스 모니터 시간 간격. 1~450 사이.
```

## 의존성

없음

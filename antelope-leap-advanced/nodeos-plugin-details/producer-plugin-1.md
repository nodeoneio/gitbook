# Prometheus Plugin

## 개요

`prometheus_plugin` 을 설정하면 `nodeos` 의 여러가지 지표를 확인할 수 있습니다.

## 사용법

다음과 같이 `config.ini` 또는 명령줄에서 사용할 수 있습니다.

```
# config.ini
plugin = eosio::prometheus_plugin [options]

# nodeos startup params
nodeos ... -- plugin eosio::prometheus_plugin [options]
```

## Prometheus Plugin 설정 옵션

다음은 `nodeos` 명령줄 또는 `config.ini` 파일에 설정할 수 있는 옵션들 입니다.

```
eosio::prometheus_plugin 설정 옵션

--prometheus-exporter-address arg (=127.0.0.1:9101)
들어오는 프로메테우스 지표 http 요청을 리스닝하는 로컬 IP 와 포트 번호
```

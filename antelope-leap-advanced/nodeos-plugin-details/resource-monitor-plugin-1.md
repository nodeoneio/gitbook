# Signature Provider Plugin

## 개요

TBD

## 사용법

다음과 같이 `config.ini` 또는 명령줄에서 사용할 수 있습니다.

```
# config.ini
plugin = eosio::signature_provider_plugin
[options]

# command-line
nodeos ... --plugin eosio::signature_provider_plugin [options]
```

## Signature Provider Plugin 설정 옵션

다음은 `nodeos` 명령줄 또는 `config.ini` 파일에 설정할 수 있는 옵션들 입니다.

```
eosio::signature_provider_plugin 설정 옵션

--keosd-provider-timeout (=5)
keosd provider 에게 서명 요청을 보낼 때 허용되는 최대 시간(ms 단위)
```

## 의존성

없음

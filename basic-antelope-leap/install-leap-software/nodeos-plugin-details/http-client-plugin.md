# HTTP Client Plugin

## 개요

`http_client_plugin` 은 `producer_plugin` 이 외부 `keosd` 인스턴스를 블록 서명자로 안전하게 사용할 수 있는 기능을 제공하는 내부 유틸리티 플러그인 입니다. `producer_plugin` 을 사용하여 블록을 생성하도록 설정한 경우에만 사용할 수 있습니다.

## 사용법

다음과 같이 `config.ini`  또는 명령줄에서 사용할 수 있습니다.&#x20;

```
# config.ini
plugin = eosio::http_client_plugin
https-client-root-cert = "path/to/my/certificate.pem"
https-client-validate-peers = 1

# command-line
nodeos ... --plugin eosio::http_client_plugin  \\
           --https-client-root-cert "path/to/my/certificate.pem"  \\
           --https-client-validate-peers 1
```

## HTTP Client Plugin 설정 옵션

다음은 nodeos 명령줄 또는 `config.ini` 파일에 설정할 수 있는 옵션들 입니다.

```
eosio::http_client_plugin 의 설정 옵션:
  --https-client-root-cert arg
    PEM으로 인코딩된, 신뢰할 수 있는 루트 인증서(또는 인증서를 포함하고 있는 파일 경로).
    TLS 연결을 확인하는 데 사용된다.(여러 번 지정할 수 있다.)
  --https-client-validate-peers arg (=1)
    true: 피어 인증서가 유효하고 신뢰할 수 있는지 검증한다.
    false: 인증서 에러를 무시한다.
```

## 의존성

* producer\_plugin

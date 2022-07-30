# HTTP Plugin

## 개요

`http_plugin` 은 nodeos 와 keosd 에서 모두 지원되는 핵심 플러그인 입니다. 이 플러그인을 사용하도록 설정하면 nodeos 또는 keosd 인스턴스에서 제공하는 RPC API 기능을 사용할 수 있습니다.

## 사용법

다음과 같이 `config.ini`  또는 명령줄에서 사용할 수 있습니다.&#x20;

```
# config.ini
plugin = eosio::http_plugin
[options]

# command-line
nodeos ... --plugin eosio::http_plugin [options]
 (or)
keosd ... --plugin eosio::http_plugin [options]
```

## HTTP Plugin 설정 옵션

다음은 nodeos 명령줄 또는 `config.ini` 파일에 설정할 수 있는 옵션들 입니다.

```
eosio::http_plugin 설정 옵션

  --unix-socket-path arg
    HTTP RPC용 유닉스 소켓을 만들기 위한 파일 이름.(data 디렉토리의 상대경로)
    사용하지 않으려면 공백으로 설정한다.
  --http-server-address arg (=127.0.0.1:8888) 
    HTTP 커넥션을 수신하기 위한 로컬 IP와 포트. 사용하지 않으려면 공백으로 설정한다.
  --https-server-address arg
    HTTPS 커넥션을 수신하기 위한 로컬 IP와 포트. 사용하지 않으려면 공백으로 설정한다.
  --https-certificate-chain-file arg
    HTTPS 커넥션을 위한 인증서 체인이 있는 파일 이름. PEM 포맷이며 https 설정에 필요.
  --https-private-key-file arg
    PEM 포맷의 HTTPS 개인 키가 있는 파일 이름. https 설정에 필요.
  --https-ecdh-curve arg (=secp384r1)
    https ECDH curve 를 사용하기 위한 설정: secp384r1 or prime256v1
  --access-control-allow-origin arg
    요청마다 반환되는 Access-Control-Allow-Origin 을 지정.
  --access-control-allow-headers arg
    요청마다 반환되는 Access-Control-Allow-Header 을 지정.
  --access-control-max-age arg
    요청마다 반환되는 Access-Control-Max-Age 을 지정.
  --access-control-allow-credentials
    각 요청에 대해 Access-Control-Allow-Credentials: true를 반환할지 여부를 지정.
  --max-body-size arg (=2097152)
    수신되는 RPC요청에 대하여 허용되는 최대 body 크기(Bytes 단위)
  --http-max-bytes-in-flight-mb arg (=500)
    http_plugin 이 http 요청을 처리하는데 사용해야 하는 최대 크기(megabytes).
    초과하면 503 error 응답.
  --http-max-response-time-ms arg (=30)
    HTTP 요청 처리에 허용되는 최대 시간.
  --verbose-http-errors
    HTTP 응답에 에러 로그를 추가.
  --http-validate-host arg (=1)
    false 로 설정되면 모든 수신되는 "Host" 헤더는 유효한 것으로 간주된다.
  --http-alias arg
    수신되는 HTTP 요청의 "호스트" 헤더에 대해 추가적으로 허용되는 값. 여러 번 지정할 수 있다. 
    디폴트로 http/https_server_address 포함.
  --http-threads arg (=2)
    https 스레드 풀에서 동작하는 worker 스레드의 수.
```

## 의존성

없음

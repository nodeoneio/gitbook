# \[Deprecated] Login Plugin

## 개요

{% hint style="warning" %}
Login Plugin 은 Leap 4.0 부터 제거되었습니다.&#x20;
{% endhint %}

`login_plugin` 은 애플리케이션이 Antelope 기반 블록체인에서 인증받을 수 있도록 지원합니다. `login_plugin` API 는 응용 프로그램에게 특정한 권한을 만족시키기 위해 계정이 서명할 수 있는지 여부를 확인시켜 줄 수 있습니다.

## 사용법

다음과 같이 `config.ini` 또는 명령줄에서 사용할 수 있습니다.

```
# config.ini
plugin = eosio::login_plugin
[options]

# command-line
nodeos ... --plugin eosio::login_plugin [options]
```

## Login Plugin 설정 옵션

다음은 nodeos 명령줄 또는 `config.ini` 파일에 설정할 수 있는 옵션들 입니다.

```
eosio::login_plugin 의 설정 옵션:
  --max-login-requests arg (=1000000)   보류중인 최대 로그인 요청 개수
  --max-login-timeout arg (=60)         보류중인 로그인 요청의 최대 타임아웃 (in seconds)
```

## 의존성

* chain\_plugin
* http\_plugin

### 의존성 로딩 예제

다음과 같이 `chain_plugin`, `http_plugin` 을 사용하도록 설정되어 있어야 합니다.

```
# config.ini
plugin = eosio::chain_plugin
[options]
plugin = eosio::http_plugin 
[options]

# command-line
nodeos ... --plugin eosio::chain_plugin [options]  \\
           --plugin eosio::http_plugin [options]
```

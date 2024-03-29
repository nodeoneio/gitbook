---
description: Basic Keosd
---

# Keosd 기초

## 개요

`keosd` 는 개인 키를 저장하고 디지털 메시지에 서명하기 위한 키 관리 서비스 데몬입니다. `keosd` 는 지갑 파일에 저장된 키를 암호화하기 위한 안전한 키 저장 매개체 역할을 합니다.

`keosd` 는 `cleos` 와 마찬가지로 Leap 소프트웨어의 일부분이기 때문에 Leap 를 설치할 때 같이 설치됩니다.

암호를 입력하여 지갑이 잠금 해제되면 `cleos` 는 `keosd` 에게 요청하여 알맞는 개인 키로 트랜잭션에 서명할 수 있습니다.

{% hint style="info" %}
`keosd` 는 Leap 노드를 운영하거나 개발 할 때만 사용하는 것을 전제로 만들어졌으므로, 그 외 상황에서는 별도의 지갑을 사용하는 것이 좋습니다.
{% endhint %}

## Keosd 사용하기

`keosd` 를 명시적으로 명령줄에서 실행할 일은 거의 없습니다. 일반적으로 `cleos` 에서 지갑과 관련된 명령을 수행하면 `keosd` 가 자동으로 실행되기 때문에 특별히 다른 방법을 사용해야 하는 이유가 없다면 이 방법으로 실행하는 것이 좋습니다.

명시적으로 `keosd` 를 실행하려면 단순히 다음과 같이 명령줄에서 옵션 없이 `keosd` 를 입력하고 실행합니다.

```
$ keosd
```

지갑 파일은 디폴트로 `~/eosio-wallet` 아래에 만들어집니다. `keosd` 의 환경설정 파일인 `config.ini` 도 이 디렉토리 안에 만들어집니다.

### Keosd 명령어 옵션

대부분의 경우 `keosd` 는 디폴트 설정으로 실행합니다. 상세한 `keosd` 명령은 `--help` 옵션을 사용하여 확인할 수 있습니다.

### 자동 잠금

자동 잠금은 `keosd` 설정에서 그나마 많이 사용하는 옵션입니다. 디폴트로 지갑은 잠금 해제된 지 15분 동안 아무 행동이 없으면 자동으로 지갑을 잠급니다. 자동 잠금 시간은 `config.ini` 파일의 `--unlock-timeout` 옵션(초 단위)에서 지정할 수 있습니다. 만약 0으로 지정하면 지갑은 항상 잠금 상태가 됩니다.

### keosd 중지

`keosd` 를 중지하려면 프로세스 목록에서 `keosd` 의 PID 를 찾아 `SIGINT` 또는 `SIGTERM` 시그널을 전달합니다.

```
$ kill -SIGINT <PID>
$ kill -SIGTERM <PID>
```

또는 간단하게 `kill <PID>` 명령으로 `keosd` 를 중지할 수 있습니다.

## Keosd 의 보안

사용자가 처음 지갑을 만들면 잠금 해제 상태가 됩니다. `keosd` 가 시작된 방식에 따라서는 프로세스를 재시작 할 때 까지 잠금 해제 상태로 남아 있을 수도 있습니다. 명시적으로 지갑을 잠그려면 다음 명령을 사용합니다.

```
$ cleos wallet lock
```

`config.ini` 설정 파일의 `--unlock-timeout` 에 설정한 시간(디폴트 15분)이 지나되거나 프로세스가 재시작되어 지갑이 잠기게 되면 패스워드를 사용하여 잠금 해제해야 합니다.

자동화된 스크립트에서 지갑을 열고 일련의 작업을 한 뒤, 작업이 끝나면 지갑을 잠그는 프로세스를 추가하는 것이 보안상 안전합니다. 다음은 지갑의 상태 플래그에 따라 지갑을 잠그거나 해제하는 파이썬 코드 예제입니다.

{% code overflow="wrap" %}
```python
def wallet_control(lock):
    if lock == True:
        subprocess.check_output([CLEOS_PATH, "wallet", "lock", "-n", WALLET_NAME],universal_newlines=True)
        return
    subprocess.check_output([CLEOS_PATH, "wallet", "unlock", "-n", WALLET_NAME, "--password", WALLET_PW],universal_newlines=True)
```
{% endcode %}

`keosd` 는 지갑을 잠그거나, 잠금 해제 하여 개인 키 저장/검색 및 디지털 메시지 서명을 할 수 있지만, 사용자 인증이나 권한 허가를 위한 기능은 가지고 있지 않습니다.

도메인 소켓을 사용하여 `keosd` 에 접근 한다면, 파일 시스템의 소켓 파일에 쓰기 권한을 가진 UNIX 사용자와 그룹은 잠금 해제된 지갑의 키를 사용하여 트랜잭션을 제출하고 `keosd` 로부터 서명된 트랜잭션을 받을 수 있습니다.

`localhost` 에 바인딩된 TCP 소켓의 경우 소유자 또는 권한에 관계없이, 모든 로컬 환경의 프로세스가 API 로 제공되는`keosd` 의 기능을 사용할 수 있습니다. 여기에는 로컬 브라우저에서 실행되는 웹 페이지의 JavaScript 코드도 포함됩니다. 단, 일부 웹 브라우저는 이에 대한 보안 기능이 있을 수 있습니다.

마찬가지로 외부 LAN/WAN 주소에 바인딩된 TCP 소켓의 경우, 원격 클라이언트가 `keosd` 를 실행하는 머신에 요청을 보낼 수 있다면 API 로 제공되는 `keosd` 의 모든 기능을 수행할 수 있습니다. 이 경우 HTTPS 로 통신이 암호화되어 있다 하더라도 큰 보안 위험을 초래할 수 있습니다. 때문에 특별한 이유가 없는 한 `keosd` 의 API 는 외부로 노출하지 않는 것이 안전합니다.

### "Failed to lock access to wallet directory; is another `keosd` running"? 메시지 해결 방법

지갑과 관련된 작업을 수행할 때 가장 흔하게 볼 수 있는 에러 메시지입니다. `cleos` 는 지갑과 관련된 명령을 실행할 때 자동으로 `keosd` 를 시작하기 때문에, 경우에 따라 같은 환경에서 여러 `keosd` 프로세스가 돌아가는 경우도 발생할 수 있습니다. 위 메시지는 이러한 경우에 나타납니다.

이 문제를 해결하려면 먼저 모든 `keosd` 데몬을 제거한 다음, 새로 `keosd` 를 시작합니다. 다음 명령을 실행하면 모든 `keosd` 데몬 프로세스를 제거할 수 있습니다.

```
pkill keosd
```

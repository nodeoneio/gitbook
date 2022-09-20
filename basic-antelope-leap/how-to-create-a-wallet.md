# 지갑 만들기

## 개요

이 단원에서 `keosd` 를 사용하여 지갑을 만드는 방법을 배울 것입니다.

## 시작하기 전에

* `cleos wallet create` 명령을 사용할 것입니다.
* `cleos` 가 설치되어 있어야 합니다. `cleos` 는 `leap` 소프트웨어를 설치할 때 `nodeos, keosd` 와 함께 설치됩니다.
* 계정이 무엇이며 블록체인에서 무슨 역할을 하는지 이해하고 있어야 합니다.
* [계정과 권한](basic-accounts-and-permissions.md)에 대한 기초적인 내용을 알고 있어야 합니다.
* 공개키와 개인키로 이루어진 비대칭 암호화 키 쌍에 대한 기초적인 이해가 필요합니다.

## 순서

지갑에 들어있는 키를 사용하려면 먼저 지갑부터 열어야 합니다. 개발 환경에서는 상관없지만 프로덕션 환경에서는 어떤 작업을 시작할 때 지갑을 열고 작업이 끝나면 바로 지갑을 닫는 것이 좋습니다. 지갑의 잠금 해제 지속 시간은 `keosd` 환경설정 파일의 `unlock-timeout` 에서 지정할 수 있습니다. 자세한 내용은 [keosd 기초](basic-keosd.md) 단원을 참조합니다.

지갑은 다음과 같이 만들 수 있습니다.

```
cleos wallet create [-n 지갑 이름] -f <패스워드를 저장할 파일 이름>
```

`-n` 옵션으로 지갑 이름을 지정하지 않으면 디폴트 지갑이 만들어지며, `-f` 옵션으로 파일 이름을 지정하지 않으면 패스워드가 콘솔 화면에 표시됩니다. 다음은 디폴트 지갑을 만들고 패스워드를 `default_wallet.pwd` 파일에 저장하는 명령입니다.

```
cleos wallet create -f default_wallet.pwd

Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
saving password to default_wallet.pwd
```

지갑을 만들고 난 직후 지갑은 열려 있는 상태입니다. 다음 명령으로 지갑 목록을 확인할 수 있습니다. \* 표시는 현재 지갑이 열려 있다는 의미 입니다.

```
cleos wallet list
Wallets:
[
  "default *"
]
```

# 계정 만들기

## 개요

본 단원에서는 `cleos` 명령을 사용하여 Antelope 블록체인의 계정을 만드는 방법을 설명합니다. 계정을 사용하면 스마트 컨트랙트를 배포할 수 있고 기타 다른 블록체인 관련 명령을 수행할 수 있습니다.

예제로 `cleos` 명령을 사용하여 디폴트 시스템 계정인 `eosio` 으로 bob 이라는 새 계정을 만들어 보겠습니다.&#x20;

## 시작하기 전에

* 본 단원을 학습하려면 `cleos` 가 설치되어 있어야 합니다. `cleos` 는 `leap` 소프트웨어를 설치할 때 `nodeos, keosd` 와 함께 설치됩니다.
* 기초적인 Antelope 블록체인의 계정과 권한에 대하여 알고 있어야 합니다.
* 기초적인 비대칭 암호화 키 쌍(공개키/개인키)에 대하여 알고 있어야 합니다.

## 순서

계정을 만들 때에는 `cleos create account` 명령을 사용합니다. 이 명령에 대한 자세한 내용은 `cleos create account --help` 명령으로 확인할 수 있습니다.&#x20;

eosio 계정으로 bob 계정을 만드는 명령은 다음과 같습니다.

```
cleos create account eosio bob EOS87TQktA5RVse2EguhztfQVEh6XXxBmgkU8b4Y5YnGvtYAoLGNN
```

위 명령의 옵션은 다음과 같습니다.

* eosio: 새 계정을 만드는 데 필요한 기존 시스템 계정 이름
* bob: 새 계정 이름 . 계정 이름은 [계정 이름 규칙 단원](protocol-guide/account-and-permissions.md)을 확인합니다.
* EOS87TQ....: 새 계정의 공개키 입니다. [키 쌍 만들기 단원](https://app.gitbook.com/s/YZT0OiBQKuAU7OjJoCgQ/\~/changes/9SOrppvOOBlShstEZa2F/basic-antelope-leap/how-to-create-key-pair)을 참조하여 만든 키를 입력합니다.

{% hint style="info" %}
Antelope 블록체인에서 새 계정을 생성하려면 새 계정을 만들 때 인증해 줄 기존 계정이 존재해야 합니다. 새로 만들어진 블록체인의 경우 디폴트 시스템 계정인 `eosio` 를 이용하여 새 계정을 만들 수 있습니다. &#x20;

디폴트 eosio 계정 active 권한의 키 쌍은 다음과 같습니다. eosio 계정으로 트랜잭션을 승인하려면 개인 키를 지갑에 미리 가져다 놓아야 합니다.

Public: EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV\
Private: 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
{% endhint %}

다음은 실행 결과의 예시입니다.

```
executed transaction: 4d65a274de9f809f9926b74c3c54aadc0947020bcfb6dd96043d1bcd9c46604c  200 bytes  166 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"bob","owner":{"threshold":1,"keys":[{"key":"EOS87TQktA5RVse2EguhztfQVEh6X...
warning: transaction executed locally, but may not be confirmed by the network yet         ]
```

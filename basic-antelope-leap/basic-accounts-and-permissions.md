---
description: Accounts and permissions
---

# 계정과 권한 기초

## 계정과 권한 기초

이 단원에서는 계정과 권한(Permission), 승인체계(Authorities)에 대하여 알아볼 것입니다.

Antelope 블록체인을 사용하려면 계정(account)이 필요합니다. 계정은 블록체인에 참여하는 참여자를 식별하기 위한 수단으로 사용되며, 스마트 컨트랙트를 배포할 때도 계정을 사용하고, 스마트 컨트랙트 트랜잭션을 승인하기 위하여 계정의 권한을 사용하기도 합니다.

이 단원에서는 다음과 같은 주요 개념을 소개할 것입니다.

* 계정(Account): 계정은 블록체인의 인증을 제어합니다. 프로토콜 가이드의 계정과 권한(준비중) 에서 계정이 어떻게 동작하는지 자세히 확인할 수 있습니다. 계정은 권한을 가지고 있고 권한은 자신만의 승인체계를 가집니다.
* 권한(Permission) 과 승인체계(Authority): 권한은 블록체인상에서 트랜잭션을 승인하는 방법을 제어합니다. 프로토콜 가이드의 권한(준비중) 에서 권한이 트랜잭션을 인증하기 위하여 어떻게 승인체계를 사용하는 지 확인할 수 있습니다.
* 키 쌍(Key pair): 암호화 공개키와 개인키의 쌍 입니다.
* cleos : nodeos 에 명령과 요청을 보낼 수 있는 명령줄 도구 입니다.
* keosd: 개인 키를 안전하게 지갑에 저장하는 로컬 저장소 입니다.
* 트랜잭션 서명: 스마트 컨트랙트 호출을 인증합니다.

이 단원은 다음과 같은 내용을 다룹니다.

* 계정과 키 쌍을 생성합니다. 이 계정들은 스마트 컨트랙트 가이드에서도 활용할 수 있습니다.
* 키 쌍 만듭니다.
* 계정의 키 쌍을 업데이트 하는 방법을 알아봅니다.

이 단원을 마치고 나면 계정의 권한과 키를 만들고 사용할 수 있습니다.

## 시작하기 전에

이 단원을 학습하려면 Antelope 블록체인 노드가 동작하고 있어야 합니다. [Nodeos 기초 단원](basic-nodeos.md)에서 Antelope 노드를 시작하는 법을 다루고 있으니 참고하여 노드를 시작할 수 있습니다.

## 계정과 권한에 대하여

블록체인 계정 이름은 사람이 읽을 수 있는 1\~12 자의 길이를 가지는 문자열입니다. 각 계정은 블록체인의 참여자와 그 참여자의 권한을 식별하기 위하여 사용됩니다. 권한은 Antelope 계정이 할 수 있는 일과 액션을 인증하는 방법을를 제어합니다.

참여자인 Alice 는 "alice" 라는 이름의 계정을 만들었습니다. 계정을 만들면은 `owner` 와 `active` 라는 두 개의 권한을 디폴트로 가집니다. 나중에 필요에 따라 Alice 에 커스텀 권한을 추가할 수도 있습니다.

이 두 권한은 다음과 같이 표기합니다.

* alice@owner
* alice@active

스마트 컨트랙트를 배포하려면 계정이 필요합니다. 스마트 컨트랙트는 반드시 계정으로 배포되어야 하며 하나의 계정은 하나의 스마트 컨트랙트 인스턴스를 소유할 수 있습니다.

## 키 쌍(Key Pair) 에 대하여

계정은 자신의 공개키와 함께 블록체인 상에 저장되며, 계정의 개인키는 keosd 와 같이 안전한 로컬 환경의 지갑에 저장됩니다. 계정은 최소한 하나의 키 쌍이 필요합니다. 블록체인은 비대칭 암호화 기술을 사용하여, 계정이 제출한 트랜잭션을 서명한 개인키가 블록체인 상에 저장된 공개키와 일치하는지를 검증합니다. 이것이 제출된 트랜잭션을 인증하는 방법입니다. `cleos` 명령은 열려 있는 `keosd` 지갑에서 자동으로 개인 키를 찾아옵니다.

## 권한과 승인체계(Permissions And Authorities)

계정은 권한을 가지고 권한은 승인체계를 가집니다. 승인체계는 권한과 연결된 승인체계 테이블(Authority Table)에 정의되어 있습니다. 어떤 계정 권한이 승인체계의 조건을 만족하고, 적절한 키로 서명되었다면 그 스마트 컨트랙트는 실행될 수 있습니다.

권한은 상속이 가능한 구조입니다. 즉 상속 관계에 있는 상위 권한은 하위 권한이 할 수 있는 모든 일을 할 수 있습니다.

`cleos` 명령에서 -p 옵션을 사용하여 트랜잭션을 서명할 때 어떤 권한을 사용할 것인지 지정할 수 있습니다. `cleos` 는 자동으로 열려 있는 지갑에서 필요한 계정과 권한의 개인키를 찾아올 것입니다.

{% hint style="info" %}
`cleos` 로 Alice 의 계정인 "alice" 의 `active` 권한을 사용하려면 명령줄에 `-p alice@active` 을 추가합니다.
{% endhint %}

* 로컬 환경의 `keosd` 에 계정과 퍼미션에 알맞는 개인 키가 저장되어 있다면 트랜잭션은 성공적으로 서명되어 블록체인에 제출될 것입니다.
* 개인 키가 블록체인에 저장된 공개키의 쌍에 해당된다면 트랜잭션이 블록체인에서 실행될 것입니다.
* 계정과 퍼미션이 트랜잭션을 실행할 권한을 가지고 있다면 트랜잭션이 실행될 것입니다. 트랜잭션의 레코드와 트랜잭션을 서명한 사용자는 영구히 블록체인상에 기록됩니다.

{% hint style="danger" %}
개인 키는 안전하게 보관해야 합니다. 개인 키는 블록체인 계정에 대한 모든 접근(full access)를 제공하기 때문에 안전한 지갑에만 보관하고 절대로 다른 사람과 공유해서는 안 됩니다.
{% endhint %}

## 지갑 만들기

[지갑 만들기 단원](https://app.gitbook.com/s/YZT0OiBQKuAU7OjJoCgQ/\~/changes/9SOrppvOOBlShstEZa2F/basic-antelope-leap/how-to-create-a-wallet)을 참조하여 지갑을 만듭니다.

## 계정 만들기

다음 순서를 따라가면 지갑과 계정을 만들 수 있습니다. 로컬 지갑이 열려 있는지를 꼭 확인한 뒤 시작해야 합니다.

1. [키 쌍 만들기](https://app.gitbook.com/s/YZT0OiBQKuAU7OjJoCgQ/\~/changes/9SOrppvOOBlShstEZa2F/basic-antelope-leap/how-to-create-key-pair)
2. [계정 만들기 ](https://app.gitbook.com/s/YZT0OiBQKuAU7OjJoCgQ/\~/changes/9SOrppvOOBlShstEZa2F/basic-antelope-leap/how-to-create-account)
3. [계정의 개인 키를 지갑으로 가져오기](https://app.gitbook.com/s/YZT0OiBQKuAU7OjJoCgQ/\~/changes/9SOrppvOOBlShstEZa2F/basic-antelope-leap/how-to-import-a-key-to-a-wallet)

계정을 만들고 나면 [계정의 정보](https://app.gitbook.com/s/YZT0OiBQKuAU7OjJoCgQ/\~/changes/9SOrppvOOBlShstEZa2F/basic-antelope-leap/how-to-get-account-information)를 확인할 수 있습니다.

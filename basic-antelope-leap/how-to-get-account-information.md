---
description: Get an Account info
---

# 계정 정보 확인하기

## 개요

이 단원에서는 계정 정보를 확인하는 방법을 배울 것입니다.

## 시작하기 전에

* `cleos` 가 설치되어 있어야 합니다. `cleos` 는 `leap` 소프트웨어를 설치할 때 `nodeos, keosd` 와 함께 설치됩니다.
* [계정과 권한](basic-accounts-and-permissions.md)에 대한 기초적인 내용을 알고 있어야 합니다.

## 순서

계정 정보를 확인하려면 `cleos get account` 명령을 사용합니다. 사용 방법은 다음과 같습니다.

```
cleos get account <계정 이름>
```

다음 예제는 시스템 계정인 `eosio` 의 정보를 얻어오는 명령과 그 결과입니다.

{% code overflow="wrap" %}
```
cleos get account eosio

created: 2018-06-01T12:00:00.000
privileged: true
permissions:
     owner     1:    1 EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
        active     1:    1 EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
memory:
     quota:       unlimited  used:     3.004 KiB

net bandwidth:
     used:               unlimited
     available:          unlimited
     limit:              unlimited

cpu bandwidth:
     used:               unlimited
     available:          unlimited
     limit:              unlimited
```
{% endcode %}

{% hint style="info" %}
블록체인 네트워크에 어떤 시스템 컨트랙트가 배포되어 있는가에 따라 표시되는 내용이 달라질 수 있습니다.
{% endhint %}

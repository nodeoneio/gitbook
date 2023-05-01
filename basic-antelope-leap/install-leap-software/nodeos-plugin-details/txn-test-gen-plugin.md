# \[Deprecated] Txn Test Gen Plugin

## 개요

{% hint style="warning" %}
Txn Test Gen Plugin 은 Leap 4.0 에서 제거되었습니다.&#x20;
{% endhint %}

이 플러그인은 트랜잭션 테스트용으로 사용합니다. 자세한 내용은 [링크](https://github.com/EOSIO/eos/blob/develop/plugins/txn\_test\_gen\_plugin/README.md)를 참조합니다.

## 사용법

다음과 같이 `config.ini` 또는 명령줄에서 사용할 수 있습니다.

```
# config.ini
plugin = eosio::txn_test_gen_plugin
[options]

# command-line
nodeos ... --plugin eosio::txn_test_gen_plugin [options]
```

## Txn Test Gen Plugin 설정 옵션

다음은 nodeos 명령줄 또는 `config.ini` 파일에 설정할 수 있는 옵션들 입니다.

```
Config Options for eosio::txn_test_gen_plugin:
  --txn-reference-block-lag arg (=0)
    트랜잭션에 대한 참조 블록을 선택할 때 헤드 블록에서 지연되는 블록 수.
    (-1 은 마지막 비가역성 블록을 의미함.)
  --txn-test-gen-threads arg (=2)
    txn_test_gen 스레드풀에서 동작하는 worker 스레드 수.
  --txn-test-gen-account-prefix arg (=txn.test.)
    계정 생성시에 사용하는 Prefix 이며 이 플러그인이 사용한다.
```

## 의존성

없음

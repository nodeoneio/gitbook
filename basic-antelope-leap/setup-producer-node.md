---
description: Set up a producer node
---

# Producer 노드 설정

## 개요

이 단원에서는 Antelope 네트워크에서 블록 생산자(BP) 노드를 설정하는 방법에 대해 설명할 것입니다. 생산자 노드라는 이름에서 알 수 있듯이 Antelope 기반 블록체인에서 블록을 생성하도록 설정되는 노드입니다. `producer_plugin` 뿐만 아니라 [다른 Nodeos 플러그인](install-leap-software/nodeos-plugin-details/)도 같이 설정해야 이 기능을 사용할 수 있습니다.

{% hint style="info" %}
BP 노드를 시작하려면 **System Contract 가 네트워크에 로드**되어 있어야 합니다. 기본 기능만을 사용하는 노드나, System Contract 를 로드하지 않은 기본 개발 노드에서는 이 단원에서 다루는 내용을 적용할 수 없습니다.
{% endhint %}

## 시작하기 전에

1. Leap 소프트웨어를 설치합니다.
2. PATH 를 설정하여 nodeos, cleos, keosd 가 어디서든 접근가능하도록 설정합니다. Leap 바이너리를 인스톨 하였다면 이 과정은 필요 없습니다.
3. 필요한 기능을 사용할 수 있도록 [Nodeos 옵션](https://developers.eos.io/manuals/eos/latest/nodeos/usage/nodeos-options)을 설정합니다.

## 단계

먼저 `config.ini` 파일을 열고 BP 등록에 필요한 정보들을 기입합니다.

### BP 이름을 설정

`config.ini` 의 `producer-name` 에 계정 이름을 설정합니다.

{% code overflow="wrap" %}
```
#config.ini:

# ID of producer controlled by this node (e.g. inita; may specify multiple times) (eosio::producer_plugin)
producer-name = <youraccount>
```
{% endcode %}

### BP의 signature-provider 정보를 설정

BP 등록용의 공개 키/개인 키 정보를 설정합니다. `cleos create key` 명령을 사용하여 새로운 BP 등록용 키 쌍을 만들 수 있습니다. 여기서 사용하는 키 쌍은 `owner/active` 키와는 다른 별개의 키 쌍을 만들어 사용하는 것이 보안상 안전합니다.이 이 키 쌍의 공개키는 BP 등록 명령을 실행할 때도 사용할 것입니다.

`signature-provider` 는 다음과 같이 3개의 필드로 구성됩니다.

* `public-key` - `regproducer` 시에 사용할 공개키.
* `provider-spec` - ":" 와 같은 형태의 문자열.
* `provider-type` - `KEY` 또는 `KEOSD` 를 설정. 일반적으로 `KEY` 를 사용.

#### **KEY 를 사용하는 경우**

{% code overflow="wrap" %}
```
# config.ini:

signature-provider = PUBLIC_SIGNING_KEY=KEY:PRIVATE_SIGNING_KEY

//Example
//signature-provider = EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV=KEY:5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```
{% endcode %}

#### **KEOSD 를 사용하는 경우**

{% code overflow="wrap" %}
```
# config.ini:

signature-provider = KEOSD:<data>   

//Example
//EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV=KEOSD:https://127.0.0.1:88888
```
{% endcode %}

### peer 목록을 입력

체인의 다른 p2p 노드 주소를 입력합니다. 여기에 입력할 주소 목록은 BP가 직접 해당 체인의 커뮤니티, 공식 웹사이트, 개발자 포털 등에서 찾아봐야 합니다.

```
# config.ini:

# Default p2p port is 9876
p2p-peer-address = 123.255.78.9:9876
```

### 필요한 플러그인을 로드

기본적으로 다음의 플러그인을 로드하고, 필요에 따라 다른 플러그인도 추가로 로딩합니다.

```
# config.ini:

plugin = eosio::chain_plugin
plugin = eosio::producer_plugin
```

### nodeos 재시작

`config.ini` 에 설정한 내용이 적용되도록 nodeos 를 재시작 합니다.

### 계정을 BP로 등록

계정이 BP가 되려면 `cleos system regproducer` 명령을 사용하여 블록 생성자로서 등록을 해야 하며 가장 기본적인 사용법은 다음과 같습니다.

```
cleos system regproducer [OPTIONS] account producer_key [url] [location]
```

* account: BP 로 사용할 Antelope 계정 이름을 입력합니다.
* producer\_key: BP 등록용 키 쌍의 공개키 입니다. 이 키 쌍은 위에서 BP 노드의 설정 파일에 기입하였습니다.
* URL: BP 의 웹사이트를 기입합니다. 이 웹 사이트에 `bp.json`, `chains.json` 과 같은 파일들이 위치하게 됩니다. 이 URL 은 로컬 네트워크를 구성할 때는 필요 없습니다.
* Location: ISO\_3166 코드를 사용합니다. (한국의 경우 410) 마찬가지로 로컬 네트워크를 구성할 때는 필요 없습니다.\
  [https://en.wikipedia.org/wiki/ISO\_3166-1\_numeric#410](https://en.wikipedia.org/wiki/ISO\_3166-1\_numeric#410)

이 명령은 디폴트로 블록 생성자 계정의 active 권한으로 실행되기 때문에, 이 권한의 개인 키가 위 명령을 실행하는 로컬 환경의 지갑에 들어 있어야 하며, 지갑도 unlock 상태이어야 합니다.

이제 명령을 실행합니다. 다음은 블록 생성자 계정 등록의 예시 입니다.

{% code overflow="wrap" %}
```bash
$ cleos system regproducer leapblockprd EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG https://www.mywebsite.io 410
```
{% endcode %}

### 등록된 BP 확인

명령이 정상적으로 수행되고 계정이 BP 로 등록되었다면 다음과 같이 `cleos system listproducers` 명령으로 확인해 볼 수 있습니다.

```
$ cleos system listproducers
Producer      Producer key                                              Url                                                         Scaled votes
...
leapblockprd  EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG                                                                 0.0000
...
```

그러나 이 계정은 아직 투표를 받지 못했기 때문에 BP 로 등록은 되었으나 활성 BP 스케줄에는 포함되어 있지 않습니다. 이 노드가 블록을 생성하게 하려면 BP 계정이 투표를 받고 활성 블록 스케줄에 포함되어야 합니다. 투표 방법은 해당 단원을 참조합니다.

만약 nodeos 의 stdout 로그를 콘솔에 출력하거나 파일에 기록하고 있다면, 등록한 BP가 투표를 받고 활성 스케줄에 포함되어 블록을 생성할 차례가 되었을 때 다음 예시와 같은 로그가 출력되는 것을 확인할 수 있습니다.

{% code overflow="wrap" %}
```
info  2022-08-22T14:34:18.906 nodeos    producer_plugin.cpp:2434      produce_block        ] Produced block 76c79aac6ddd6f3b... #3 @ 2022-08-22T14:34:19.000 signed by leapblockprd [trxs: 0, lib: 2, confirmed: 0]
```
{% endcode %}

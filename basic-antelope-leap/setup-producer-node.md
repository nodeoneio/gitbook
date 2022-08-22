# Producer 노드 설정

### 개요

이 단원에서는 Mandel/EOSIO 네트워크에서 블록 생산자(BP) 노드를 설정하는 방법에 대해 설명할 것입니다. 생산자 노드라는 이름에서 알 수 있듯이 Mandel/EOSIO 기반 블록체인에서 블록을 생성하도록 설정되는 노드입니다. `producer_plugin` 뿐만 아니라 [다른 Nodeos 플러그인](install-leap-software/nodeos-plugin-details/)도 설정하여 이 기능을 사용할 수 있습니다.

{% hint style="info" %}
BP 노드를 시작하려면 **System Contract 가 네트워크에 로드**되어 있어야 합니다. 기본 기능만을 사용하는 노드나, System Contract 를 로드하지 않은 기본 개발 노드에서는 이 단원에서 다루는 내용을 적용할 수 없습니다.
{% endhint %}

### 시작하기 전에

1. Mandel 소프트웨어를 설치합니다.
2. nodeos, cleos, keosd 가 어디서든 접근가능하도록 설정한다. 쉘 스크립트로 Mandel 을 빌드한다면 [Install Script](https://developers.eos.io/manuals/eos/latest/install/build-from-source/shell-scripts/install-eosio-binaries)를 사용합니다.
3. 필요한 기능을 사용하기 위하여 [Nodeos 옵션](https://developers.eos.io/manuals/eos/latest/nodeos/usage/nodeos-options)을 설정합니다.

### 단계

#### 계정을 BP로 등록

계정이 BP가 되려면 `cleos system regproducer` 명령을 사용하여 블록 생성자로서 등록을 해야 합니다.&#x20;

`cleos create key` 명령을 사용하여 블록 생성자 등록용 키 쌍을 생성하고 공개 키를 BP 등록시 사용합니다. 이 때 사용하는 키 쌍은 `owner/active` 키와는 다른 별개의 키 쌍을 사용하는 것이 보안상 안전합니다.

Location 은 ISO\_3166 코드를 사용합니다. (한국은 410)\
[https://en.wikipedia.org/wiki/ISO\_3166-1\_numeric#410](https://en.wikipedia.org/wiki/ISO\_3166-1\_numeric#410)

```bash
cleos system regproducer accountname1 EOS1234534... <https://www.nodeone.io> 410
```

#### BP 이름을 설정

`config.ini` 의 `producer-nam`e 에 계정 이름을 설정합니다.

```
# config.ini:

# ID of producer controlled by this node (e.g. inita; may specify multiple times) (eosio::producer_plugin)
producer-name = <youraccount>
```

#### BP의 signature-provider 정보를 설정

BP의 공개 키/개인 키 정보를 설정합니다. 이 키 쌍은 위에서 BP 를 등록할 때 사용했던 키 쌍입니다.

`signature-provider` 는 다음과 같이 3개의 필드를 정의해야 합니다.

* `public-key` - `regproducer` 시에 사용했던 Mandel 공개키
* `provider-spec` - ":" 와 같은 형태의 문자열
* `provider-type` - KEY 또는 KEOSD를 설정. 일반적으로 KEY 를 사용합니다.

**KEY 를 사용하는 경우**

```
# config.ini:

signature-provider = PUBLIC_SIGNING_KEY=KEY:PRIVATE_SIGNING_KEY

//Example
//signature-provider = EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV=KEY:5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```

**KEOSD 를 사용하는 경우**

```
# config.ini:

signature-provider = KEOSD:<data>   

//Example
//EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV=KEOSD:https://127.0.0.1:88888
```

#### peer 목록을 입력

체인의 다른 p2p 노드 주소를 입력합니다. 여기에 입력할 주소 목록은 BP가 직접 해당 체인의 커뮤니티, 공식 웹사이트, 개발자 포털 등에서 찾아봐야 합니다.

```
# config.ini:

# Default p2p port is 9876
p2p-peer-address = 123.255.78.9:9876
```

#### 필요한 플러그인을 로드

기본적으로 다음의 플러그인을 로드하고, 필요에 따라 다른 플러그인도 추가로 로딩합니다.

```
# config.ini:

plugin = eosio::chain_plugin
plugin = eosio::producer_plugin
```

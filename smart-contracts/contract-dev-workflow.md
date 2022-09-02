# 스마트 컨트랙트 개발의 기본 흐름

## 토큰 배포, 발행 및 전송: eosio.token 스마트 컨트랙트 사용법

Leap 소프트웨어를 설치하였습니다. 이제는&#x20;

이 단원에서는 스마트 컨트랙트를 다루는 가장 기본적인 내용으로, 토큰의 개념과 `eosio.token` 스마트 컨트랙트, 그리고 cleos 를 사용하여 `eosio.token` 의 액션을 호출하는 방법을 배울 것입니다.

### 단계1: 스마트 컨트랙트 소스 받아오기

원하는 위치에 contracts 디렉토리를 만들고 이동합니다.

```jsx
mkdir CONTRACTS_DIR
cd CONTRACTS_DIR
```

github 저장소에서 eosio.contracts 소스를 다운로드 받습니다.

```jsx
https://github.com/eosnetworkfoundation/eos-system-contracts.git
```

이 저장소에는 여러가지 스마트 컨트랙트 소스가 저장되어 있는데, 그 중 eosio.token 컨트랙트가 본 섹션에서 다룰 내용입니다.

이제 eosio.contracts/contracts/eosio.token 으로 이동합니다.

```jsx
cd eosio.contracts/contracts/eosio.token
```

### 단계2: 스마트 컨트랙트를 위한 계정 생성

토큰 계약을 배포하기 전에 이를 배포하기 위한 계정을 생성합니다. 이 계정은 일반에 공개되어 있는 eosio 개발용 키를 사용할 것입니다. 이 키는 절대로 프로덕션 용으로 써서는 안 됩니다.

```jsx
cleos create account eosio eosio.token EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
```

### 단계3: 스마트 컨트랙트 컴파일

다음 명령으로 스마트 컨트랙트를 컴파일 합니다. 성공적으로 컴파일하면 eosio.token.wasm 파일이 생성됩니다.

```jsx
eosio-cpp -I include -o eosio.token.wasm src/eosio.token.cpp --abigen
```

### 단계4: 토큰 컨트랙트 배포

이제 배포할 준비가 되었습니다. 다음 명령으로 토큰 컨트랙트를 배포합니다.

```jsx
$ cleos set contract eosio.token CONTRACTS_DIR/eosio.contracts/contracts/eosio.token --abi eosio.token.abi -p eosio.token@active

Reading WASM from ...eosio.contracts/contracts/eosio.token/eosio.token.wasm...
Publishing contract...
executed transaction: a68299112725b9f2233d56e58b5392f3b37d2a4564bdf99172152c21c7dc323f  6984 bytes  6978 us
#         eosio <= eosio::setcode               {"account":"eosio.token","vmtype":0,"vmversion":0,"code":"0061736d0100000001a0011b60000060017e006002...
#         eosio <= eosio::setabi                {"account":"eosio.token","abi":"0e656f73696f3a3a6162692f312e310008076163636f756e7400010762616c616e63...
warning: transaction executed locally, but may not be confirmed by the network yet         ]
```

### 단계5: 토큰 만들기

배포가 성공했으면 토큰을 만들어 봅시다. eosio.token 의 create action 에 적절한 매개변수를 넘겨 호출하면 새로운 토큰을 만들 수 있습니다. 이 액션은 다음과 같은 내용을 가지는 하나의 인수를 필요로 합니다.

* 발행자(issuer)가 되는 eosio 계정: 여기서 alice 라는 계정을 사용하겠습니다. 발행자는 issue 액션을 호출할 수 있는 권한을 가지며 또한 closing account 나 retiring token과 같은 액션을 수행할 수도 있습니다.
* asset 타입:  이 정보는 두 가지 데이터로 구성됩니다. 첫 번째 인자인 부동소수점 숫자는 토큰의 최대 공급량(maximum supply)이며, 두 번째 인자는 토큰의 심볼을 나타내는 대문자 알파벳(티커)입니다. \
  예) "1.0000 SYS"

아래는 위치 인자(positional argument) 를 이용한 호출 방법의 예제 입니다.

```jsx
cleos push action eosio.token create '[ "alice", "1000000000.0000 SYS"]' -p eosio.token@active
```

위 명령으로 소수점 4자리의 정밀도를 가지고, 최대 공급량이 100000000.0000 인 새 토큰 SYS를 만들었습니다. 그리고 alice 도 발행인으로 지정했습니다. 또한 스마트 컨트랙트가 이 토큰을 만들기 위해 eosio.token 의 권한을 필요로 하기 때문에 -p eosio.token@active 옵션으로 권한을 전달하였습니다.

액션 호출시에 다음과 같이 각 인수에 명확한 이름을 지정하는 형식으로 사용 할 수도 있습니다.

```jsx
cleos push action eosio.token create '{"issuer":"alice", "maximum_supply":"1000000000.0000 SYS"}' -p eosio.token@active

executed transaction: 10cfe1f7e522ed743dec39d83285963333f19d15c5d7f0c120b7db652689a997  120 bytes  1864 us
#   eosio.token <= eosio.token::create          {"issuer":"alice","maximum_supply":"1000000000.0000 SYS"}
warning: transaction executed locally, but may not be confirmed by the network yet         ]
```

### 단계6: 토큰 발행

이제 발행자(issuer)인 alice 는 새 토큰을 발행할 수 있게 되었습니다. 앞서 설명하였듯이 토큰 발행은 발행자만이 할 수있습니다. 따라서 issue 액션을 호출할 때 반드시 -p alice@active 권한을 같이 넘겨야 합니다.

```jsx
$ cleos push action eosio.token issue '[ "alice", "100.0000 SYS", "memo" ]' -p alice@active

executed transaction: d1466bb28eb63a9328d92ddddc660461a16c405dffc500ce4a75a10aa173347a  128 bytes  205 us
#   eosio.token <= eosio.token::issue           {"to":"alice","quantity":"100.0000 SYS","memo":"memo"}
warning: transaction executed locally, but may not be confirmed by the network yet         ]
```

트랜잭션을 검사하려면 "브로드캐스트 안 함" 및 "트랜잭션을 json으로 반환"을 나타내는 -d 와 -j 옵션을 사용하면 됩니다. 이 옵션은 개발 중에 사용하면 유용합니다.

```jsx
cleos push action eosio.token issue '["alice", "100.0000 SYS", "memo"]' -p alice@active -d -j
```

### 단계7: 토큰 전송

이제 alice 가 토큰을 발행했으니 다른 계정 bob으로 토큰을 전송해 보겠습니다.

```jsx
$ cleos push action eosio.token transfer '[ "alice", "bob", "25.0000 SYS", "m" ]' -p alice@active

executed transaction: 800835f28659d405748f4ac0ec9e327335eae579a0d8e8ef6330e78c9ee1b67c  128 bytes  1073 us
#   eosio.token <= eosio.token::transfer        {"from":"alice","to":"bob","quantity":"25.0000 SYS","memo":"m"}
#         alice <= eosio.token::transfer        {"from":"alice","to":"bob","quantity":"25.0000 SYS","memo":"m"}
#           bob <= eosio.token::transfer        {"from":"alice","to":"bob","quantity":"25.0000 SYS","memo":"m"}
warning: transaction executed locally, but may not be confirmed by the network yet         ]
```

이번에는 3개의 transfer 액션이 출력된 것을 볼 수 있습니다. 출력된 내용은 호출된 모든 액션 핸들러와 그들이 호출된 순서, 그리고 액션에 의해 생성된 출력이 있으면 표시합니다.

이제 bob이 토큰을 받았는지 확인해 보겠습니다.

```jsx
cleos get currency balance eosio.token bob SYS

25.0000 SYS
```

alice 의 잔고도 확인해 보겠습니다.

```jsx
cleos get currency balance eosio.token alice SYS

75.0000 SYS
```

이렇게 eosio.token 스마트 컨트랙트를 컴파일하고 배포한 뒤 컨트랙트 내부의 액션을 호출하는 방법을 알아보았습니다.

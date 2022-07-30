---
description: Under the hood of a smart contract
---

# 스마트 컨트랙트의 뒤에서 일어나는 일

효율적인 스마트 컨트랙트를 작성할 수 있으려면 뒤에서 무슨 일이 일어나고 있는지 이해해야 합니다. 경험이 풍부한 개발자는 마이크로컨트롤러 또는 임베디드 장치 프로그래밍과 많은 유사점을 발견할 것입니다. 여기에는 특정한 제약 조건이 있고, 여러분이 설계한 응용 프로그램이 그것들에 적합해야 합니다. 프로그램은 라이브러리와 리소스를 효율적으로 활용해야 합니다. 또한 확장성이 뛰어나, 배포 전에 수십 개가 아닌 수만 개의 데이터 항목을 가지고 테스트 했을 때 막히지 않아야 합니다.&#x20;

스마트 계약 운영에는 데이터 핸들링과와 실행 환경이라는 두 가지 중요한 측면이 있습니다

## 데이터 직렬화(Data Serialisation)

일반적으로 직렬화라는 용어는, 나중에 발생할 역직렬화를 위해 데이터 구조를 바이트 시퀀스로 변환될 때 사용됩니다. JSON 및 Google Protocol Buffers가 직렬화의 일반적인 예입니다.&#x20;

JSON 과 Protocol Buffers 는 모두 매우 유연하며 많은 응용 프로그램에서 사용됩니다. 그러나 일부 복잡한 파싱 및 상태 추적(예: JSON의 중괄호 및 구문 요소)과 오류 처리가 필요합니다. EOSIO는 빠른 트랜잭션 실행을 위해 설계되었기 때문에 유연성 제한적인 직렬화 표준을 가지게 되지만 대신 스마트 컨트랙트 내에서 훨씬 더 빠른 처리가 가능합니다.&#x20;

직렬화 형식에는 필드 구분자가 없으며 기본 형식에는 길이 필드가 없습니다. 따라서 작성한 측과 읽는 측 모두 직렬화 또는 역직렬화에 앞서 데이터 구조에 대한 공통적인 이해가 필요합니다.&#x20;

C++ 스마트 컨트랙트가 직렬화된 데이터 구조를 읽거나 써야 하는 경우, C++ 스마트 컨트랙트에 대한 구조체 타입 정의가 있어야 합니다. CDT(EOSIO Contract Development Toolkit)는 이 프로세스를 자동화하여 개발자가 알아차리지 못하게 하는 다양한 방법을 제공합니다. 때로는 개발자가 전체 프로세스를 이해하지 못하여 혼란으로 이어지기도 합니다.&#x20;

소스 코드에 \[\[eosio::table]] 속성 지정자를 사용하여 CDT에 구조체를 직렬화 및 역직렬화하기 위한 C++ 메서드를 준비하도록 지정할 수 있습니다.

예를들면,

```
struct [[eosio::table("stock")]] stockrow {
  uint64_t       skuid;
  uint64_t       items_onsale = 0; // number of items available on sale
  time_point     last_sale;
  auto primary_key()const { return skuid; }
};

typedef eosio::multi_index<name("stock"), stockrow> stockrows;
```

이 구조체 정의는 세 개의 직렬화된 64비트 정수와 메모리의 구조 사이를 변환하는 reader 및 writer 메소드를 만듭니다. 이 바이트 배열이 다른 구조 정의에 대해 작성된 경우 reader 메서드가 의미 없는 정보를 읽거나 배열의 바이트 수가 예상한 것과 다르기 때문에 오류가 발생합니다. 따라서 reader 에 필드 순서가 다르거나, 다른 유형(예: 64비트 대신 32비트 정수)이 있거나 또는 필드 수가 다른 구조체 정의가 있는 경우, reader 가 해당 바이트 배열에 저장한 내용을 읽을 수 없습니다.&#x20;

액션 정의는 완전히 다르게 보이지만 작업 자체는 유사합니다.

```
[[eosio::action]] challenge(name challenger, uint64_t id, 
                             name account, checksum256 hash, 
                             uint32_t expire_seconds)
 {
```

액션 인수도 동일한 직렬화 프로토콜을 사용하여 직렬화 및 역직렬화되므로 C++ 컴파일러는 바이트 배열에서 인수를 패키징하거나 언패키징하는 방법을 알아야 합니다. \[\[eosio::action]] 속성(단순하게 표현하기 위한 ACTION 매크로도 있음)은 컴파일러에 함수 인수가 직렬화되거나 역직렬화됨을 알려줍니다. 위의 예제에서 액션 인수는 바이트 배열로 구성됩니다. 여기서 세 개의 64비트 정수는 32바이트 배열 뒤에 이어집니다. 이 모든 값은 고정 길이를 가지므로 두 값 사이에 구분 기호가 없고 길이에 대한 정보도 없습니다. 따라서 본질적으로 구조체나 함수 인수를 처리하는 데 차이가 없습니다.&#x20;

구조에 std::string 또는 std::vector와 같은 가변 길이 필드가 있는 경우 직렬화 프로토콜은 데이터 시작 부분에 데이터의 길이를 추가합니다. 길이는 변수 길이의 특수한 유형인 varuint로 저장되므로 문자열이 짧을수록 길이 필드도 짧아집니다.&#x20;

경우에 따라 CDT는 eosio::public\_key 타입의 필드를 포함하는 것과 같은 복잡한 데이터 구조를 직렬화하는 메서드를 만들 수 없습니다. 이러한 경우 컨트랙트 코드에는 EOSLIB\_SERIALIZE 매크로를 사용하여 직렬화 방법을 명시적으로 정의해야 합니다.

```
struct [[eosio::table("messages")]] message {
  uint64_t         id;             /* autoincrement */
  name             sender;
  vector<char>     iv;
  public_key       ephem_key;
  vector<char>     ciphertext;
  checksum256      mac;
  auto primary_key()const { return id; }
};
EOSLIB_SERIALIZE(message, (id)(sender)(iv)(ephem_key)(ciphertext)(mac));
typedef eosio::multi_index<name("messages"), message> messages;  
```

스마트 컨트랙트 외부 프로그램이 직렬화된 데이터를 읽거나 써야 하는 경우 일반적으로 스마트 컨트랙트와 함께 공개되는 ABI(Application Binary Interface)를 사용합니다. EOSIO ABI는 블록체인에 저장하기에 적합한 컴팩트 바이너리 형태와 인간 및 기계가 읽기에 적합한 JSON의 두 가지 방식으로 표현될 수 있습니다. ABI는 스마트 컨트랙트에 정의된 모든 구조체(와 구조체로서의 액션 인수)를 반영합니다. 지갑이나 다른 블록체인 클라이언트와 같은 외부 프로그램은 보통 ABI를 먼저 읽고, 스마트 컨트랙트 테이블 내용을 읽거나, 트랜잭션을 전송하기 전에 액션 인수를 인코딩할 수 있습니다.

## 액션 디스패처(Action dispatcher)

C++ CDT를 사용하여 간단한 스마트 컨트랙트를 작성하는 경우 일반적으로 클래스를 정의한 다음 특정 메서드를 액션으로 표시합니다.&#x20;

그러나 작성된 스마트 컨트랙트를 실행하는 nodeos 는 C++ 클래스 및 메서드에 대해 전혀 알지 못합니다. nodeos 가 하는 일은 새로 초기화된 WASM 가상 시스템 내에서 apply() 함수(WASM 바이너리에서 내보낸 C 함수)를 호출하고 다음과 같은 여러 인수를 전달하는 것입니다.

`void apply( uint64_t receiver, uint64_t code, uint64_t action )`

이는 C 함수이며, eosio::name 오브젝트를 이용하여 사용하지만 사실 이 인수는 정수 값입니다. CDT 헤더 파일이 name 을 정수로 변환하는 것입니다. receiver는 초기 액션을 받은 계정 이름이고, code는 현재 특정 시점에 액션을 실행하는 계정입니다.&#x20;

어떤 액션이 트랜잭션 입력이거나 eosio::action::send() 실행의 결과인 경우 code 는 receiver 와 동일한 계정입니다. 그렇지 않은 경우, 이 호출은 require\_recipient() 호출의 결과이며, receiver 는 require\_recipient()가 호출된 계약의 계정 이름을 보유하므로 code 계정과 다릅니다.&#x20;

기본적으로 code 와 receiver 를 비교하여 원래의 액션 호출 또는 알림을 처리하고 있는지 확인할 수 있습니다.&#x20;

action 인수는 액션 이름에 해당합니다. require\_recipient()의 결과를 처리하는 경우 action은 require\_recipient()가 호출된 원래 액션의 이름입니다. 그렇지 않은 경우 이는 컨트랙트에서 호출된 액션의 이름입니다.

따라서 apply() 함수 내의 디스패처는 다음과 같은 몇 가지 작업을 수행해야 합니다.&#x20;

1. code 와 receiver 를 비교하여 액션 호출인지 알림인지 확인합니다.&#x20;
2. 이 액션 또는 알림에 대한 핸들러를 찾습니다.&#x20;
3. 액션 인수를 디코딩하여 핸들러에 전달합니다.&#x20;

보시다시피 action 인수는 apply() 함수에 직접 전달되지 않습니다. 실행 환경은 이를 위해 두 가지 저수준의 C 호출을 제공합니다.&#x20;

1. action\_data\_size()는 직렬화된 인수의 길이를 알려줍니다.
2. read\_action\_data()는 직렬화된 인수를 제공된 버퍼에 복사합니다.&#x20;

직렬화된 액션 인수는 구조체에 대한 정보를 가지고 있지 않습니다. 클라이언트가 패킹한 것이 무엇이든 간에 바이트 벡터를 전달받습니다. 이제 디스패처는 이 데이터에서 개별 필드를 추출하여 핸들러에게 전달해야 합니다. 그리고 그것이 가지고 있는 유일한 정보는 컨트랙트 소스에 있는 클래스 메소드 정의입니다.&#x20;

여기서 C++는 템플릿을 사용하면 편리합니다. 디스패처 코드는 클래스 메서드를 받는 일반화된 템플릿이며, 타입 목록으로서 인수 목록을 역직렬화기(deserializer)로 전달합니다.&#x20;

따라서 호출자가 잘못된 순서로 인수를 직렬화했거나 잘못된 데이터 유형(예를 들어, 디스패처가 uint32를 예상했지만 실제로는 uint64 를 패킹)을 사용하는 경우, 발송자가 인수를 직렬화하지 못하기 때문에 액션이 핸들러로 전달되기도 전에 실패할 수 있습니다.&#x20;

최신 CDT 키트를 사용하면 CDT가 \[\[eosio::action]] 및 \[\[eosio:on\_notify]] 로 라벨링 된 C++ 소스에서 apply() 함수를 생성하므로 신경 쓸 필요가 없습니다.

위의 challenge 액션 예제를 자세히 살펴보겠습니다. \[\[eosio::action]] 에는 이름 지정자가 없으므로 \[\[eosio::action("challenge")]] 와 동일합니다. paren 에 있는 이름은 액션 이름을 정의하고 이는 디스패처가 사용하며 동시에 ABI에 공개 됩니다. 그런 다음 클래스 메서드 이름이 나오는데, 이 이름은 이론적으로 액션 이름과 다를 수 있습니다. 그러나 실제로는 메소드의 이름이 액션 이름과 같다면 더 편리합니다.&#x20;

통지 핸들러는 다음과 같이 특정 컨트랙트 이름에 특정되거나 와일드카드를 사용하는 일반 계약 이름일 수 있습니다.

```
/* specific handler reactinhg on transfers in system token only */
[[eosio::on_notify("eosio.token::transfer")]] 
void on_payment (name from, name to, asset quantity, string memo) {
  ...
}

/* generic handler catching transfers in every token contract */
[[eosio::on_notify("*::transfer")]] 
void on_transfer (name from, name to, asset quantity, string memo) {
  ...
}
```

두 경우 모두 다 디폴트 디스패처가 일치하는 핸들러를 찾을 것이며 그에 알맞게 알림 인수를 역직렬화 할 것입니다. [스마트 컨트랙트 보안 챕터](https://cc32d9.gitbook.io/eosio-smart-contract-developers-handbook/design-guidelines/smart-contract-security)에서 이러한 핸들러의 잠재적 보안 리스크를 확인할 수 있습니다.

## 액션 호출하기

RPC 클라이언트가 트랜잭션을 패킹하는 경우, 컨트랙트의 ABI 만이 해당 인수에 대하여 알 수 있는 유일한 정보 입니다.

그러나 eosio::action::send()를 사용하여 컨트랙트 내부에서 어떤 액션을 호출하는 경우에는 ABI가 없습니다.  ABI를 조회하거나 호출된 컨트랙트의 소스 코드를 읽어봄으로써 어떤 인수가 있고 그 타입이 무엇인지을 추정하는 것만이 가능합니다(가능한 경우에만). 호출된 컨트랙트가 해당 액션의 정의를 변경할 경우 호출된 컨트랙트 코드를 업데이트해야 해당 액션을 다시 호출할 수 있습니다.

## 액션 내부에서 보여지는 트랜잭션

일단 트랜잭션 실행 중에 스마트 계약이 통제권을 잡으면, 액션 호출 전에 무슨 일이 있었는지에 대한 특정 정보를 얻어낼 수 있지만, 모든 것은 알아낼 수 있는 것은 아닙니다.

[트랜잭션 생애주기](https://cc32d9.gitbook.io/eosio-smart-contract-developers-handbook/how-eosio-works/life-cycle-of-a-transaction)에 설명된 대로 트랜잭션 실행은 일련의 WASM 가상 시스템으로 구성되며, 코드를 가지고는 현재 사용자가 특정 순간에 어떤 위치에 있는지 확인할 수 없습니다. 액션 핸들러는 최상위 액션 목록에서 호출되거나 다른 액션 실행의 결과로 호출될 수 있습니다.&#x20;

우리가 알 수 있는 것은 블록체인에 제출된 초기 트랜잭션의 내용입니다. 이는 트랜잭션 바이트를 제공된 버퍼에 복사하는 내장 메소드입니다. 또한 CDT는 트랜잭션 및 액션 클래스를 제공하여 필요에 따라 트랜잭션 내용을 디코딩하고 역직렬화할 수 있도록 지원합니다. 그러나 이 방법으로는 처음에 호출된 컨트랙트 및 액션과, 실행을 승인한 계정 목록만이 표시됩니다.

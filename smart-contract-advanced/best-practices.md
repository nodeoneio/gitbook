# 모범 사례

## 명명 규칙(Naming Convention)

EOSIO 스마트 컨트랙트를 구현할 때 혹은 EOSIO 블록체인에 데이터를 저장할 때, 계정인 액션 테이블 등에 대하여 EOSIO 명명 규칙을 따르는 것이 좋습니다.

### EOSIO 이름

* 인코딩 된 모든 EOSIO 이름을 적용한다.(계정, 액션, 테이블 등)
* 블록체인 에서 64비트 부호 없는 정수(uint64\_t)로서 인코딩 됨.
* . , 1-5 , a-z 를 사용하여 base32로 인코딩 된 첫 12 개의 문자를 가진다.
* . , 1-5 , a-j 를 사용하여 base16로 인코딩 된 13 번째 문자를 가진다.

### 표준 계정 이름

* . , 1-5 , a-z 의 base32 셋으로 작성된, 정확하게 12개의 문자를 가져야 한다.
* 문자 수가 12개를 넘기거나 그보다 적어서는 안 된다.
* 소문자 a-z 로 시작헤야 한다.
* 마지막 문자로 마침표(.) 를 쓸 수 없다.

### 비 표준 계정 이름

* . , 1-5 , a-z 의 base32 셋으로 작성된, 1 \~ 12개의 문자를 가질 수 있다.
* 문자 수를 12개 이상 사용할 수 없다.
* 마지막 문자로 마침표(.) 를 쓸 수 없다.

### 테이블, 구조체, 클래스, 함수(액션) 이름

* 1 \~ 13 개의 문자로 구성할 수 있다.
* . , 1-5 , a-z 를 사용하여 base32로 인코딩 된 첫 12 개의 문자를 가진다.
* . , 1-5 , a-j 를 사용하여 base16로 인코딩 된 13 번째 문자를 가진다.

### 형식(format)

아래 그림은 64비트 부호 없는 정수로 이루어진 12문자의 문자열 형식이다. 만약 13번째 문자가 포함된다면 1 문자당 2^4 = 16 개의 케이스를 포함한다.(1 (.) + 5 (1-5) + 10 (a-j))

![](<../.gitbook/assets/image (3).png>)

### 인코딩/디코딩

EOSIO name 오브젝트는 eosio::name 클래스를 사용하여 생성하고 인코딩/디코딩 할 수 있습니다.

1. std::string 문자열을 EOSIO 이름 개체로 인코딩 하려면 eosio:name() 생성자를 사용한다.
2. char\* 리터럴 문자열을 EOSIO 이름 개체로 인코딩 하려면 ""\_n 연산자를 사용한다.
3. EOSIO 이름 개체를 std::string 으로 디코딩하려면, eosio::to\_string() 함수를 사용한다.

### 예제

```cpp
auto eosio_user = eosio::name{user};  //user 문자열을 eosio::name 개체로 인코딩
auto user_str = user_name_obj.to_string(); //eosio::name 개체를 문자열로 디코딩
auto standard_account = "standardname"_n;  //리터럴 문자열을 eosio::name 으로 인코딩
auto non_standard_account = ".standard"_n; //리터럴 문자열을 eosio::name 으로 인코딩
```

## 자원 계획

RAM은 얼마나 필요할까요? 대답하기 쉽지 않은 질문이고, 완벽한 답도 없습니다. 블록체인 애플리케이션이 얼마나 빠르게, 얼마나 성장할지에 대한 예측에 따라 컨트랙트의 액션을 측정하고 그에 따라 계획을 수립함으로써 이 문제를 파악해 둘 필요가 있습니다. 블록체인 애플리케이션이 늘어남에 따라 더 많은 스토리지 용량이 필요한 경우 RAM을 더 구입해야 할 것입니다. 3일의 스테이킹 기간 동안 더 많은 액션을 실행해야 하는 경우 CPU 대역폭에 더 많은 토큰을 할당해야 할 것입니다. 블록 체인 응용 프로그램이 커짐에 따라 더 많은 액션이 블록체인에 저장되는 경우, 그만큼 더 많은 토큰을 NET 대역폭에 할당하여 NET 대역폭 최대 제한을 확장해야 합니다.

네, 좋습니다. 그런데 얼마나 해야 할까요?

블록체인 애플리케이션에 적용되는 다양한 비즈니스 시나리오를 테스트하고 시뮬레이션하고 리소스 사용량을 측정해야 합니한다. 그 때문에, 퍼블릭 테스트 네트워크가 존재하는 것이다. 여기서 각 액션에서 사용하는 RAM, CPU 및 NET 양을 측정하고 최악의 경우 및 최적의 경우에 대비한 비즈니스 시나리오를 측정할 수 있습니다. 그런 다음에야 블록체인 애플리케이션의 리소스 요구 사항을 제대로 파악하고 구축할 수 있을 것입니다.

스마트 컨트랙트, 블록체인 애플리케이션 및 사용자 기반(user base)이 어떻게 퍼블릭 테스트넷에서 블록체인 리소스를 사용하고 있는지 정확히 파악하면 퍼블릭 또는 프라이빗 EOSIO 기반 네트워크에서 무엇을 시작해야 할 지 계획할 수 있습니다. 다른 애플리케이션과 마찬가지로 이 시점부터는 애플리케이션 성능에 대한 통계 및 지표를 알려주는 모니터링 도구를 사용하는 것이 좋습니다.

물론 각 네트워크가 시스템 컨트랙트를 수정하여 네트워크마다 일부 다른 부분이 있을 수도 있습니다. EOSIO 코드는 오픈 소스이며, 각 네트워크의 요구사항에 맞게 조정할 수 있습니다. 테스트 중인 네트워크의 경우 이러한 차이를 알고 이를 고려하여 설계해야 합니다.

EOSIO 커뮤니티는 또한 이러한 노력에 도움을 줄 수 있는 도구를 제공하고 있습니다. 한 예로 [https://www.eosrp.io을](https://www.eosrp.xn--io-tv8j) 들 수 있습니다. RAM 가격이 다르고 CPU 및 NET 대역폭 할당도 다 다르기 때문에, 이전 단원에서 설명한 것처럼 이 도구를 사용하면 특정 양의 토큰 및 리소스의 양을 추정할 수 있으며, 그 반대도 가능합니다.

리소스 계획의 또 다른 측면은 컨트랙트가 효율적인지 확인하는 것, 즉 리소스를 불필요하게 사용하지 않도록 하는 것입니다. 따라서 자신만의 스마트 컨트랙트 및 블록체인 애플리케이션을 작성할 때 다음 질문에 대한 답을 찾는 것이 좋습니다.

* 스마트 컨트랙트는 블록 체인에 저장해야 하는 정보만 저장하고 나머지는 데이터 저장을 위한 대체 수단(예: IPFS)을 사용하고 있는가?
* 다수의 스마트 컨트랙트가 존재 할 때, 인라인 액션을 통해 서로 너무 많이 통신하고 있지는 않은가? 일부 스마트 컨트랙트를 하나로 병합하면 컨트랙트간의 인라인 액션을 생성하지 않아도 되므로 전체 인라인 액션 수가 감소하여 리소스 소비량이 감소할 수 있다.
* RAM 비용의 일부를 고객이 지불하도록 스마트 컨트랙트를을 변경할 수 있는가? 주소록 컨트랙트에서 RAM에 추가된 각각의 신규 계정이 개별 데이터를 저장하는 데 필요한 금액을 얼마나 지불하게 되었는지 기억하고 있는가?
* 또는 반대로, 스마트 컨트랙트에서 사용을 금지하는 액션에 접근할 때 고객이 RAM이나 CPU를 너무 많이 지불하도록 하고 있지는 않는가? 블록체인 애플리케이션의 성장과 성공을 위해 이러한 비용 중 일부를 부담하는 것이 더 나을 수도 있을까?

## 데이터 설계와 마이그레이션

EOSIO 기반 블록체인을 통해 개발자는 스마트 계약 코드를 쉽게 업데이트할 수 있습니다. 그러나 데이터 업데이트 및 마이그레이션과 관련하여 고려해야 할 사항이 몇 가지 있습니다. 다중 인덱스 테이블 API는 블록체인 상태를 저장하고 업데이트하는 메커니즘 중 하나입니다. 이는 키-값 API는 다른 것입니다. 다중 인덱스 테이블 API는 RAM 에다 데이터 구조를 만들고 사용합니다. 데이블이 블록체인에 생성되고 배포된 후 이러한 구조를 업데이트 하려는 경우 제한이 있습니다. 아래에는 스마트 컨트랙트의 데이터 설계 및 업데이트와 데이터 마이그레이션에 대한 몇 가지 접근 방식이 나와 있습니다.

### 다중 인덱스 테이블의 구조를 어떻게 변경하는가

아래 소개된 전략 중에 필요에 따라 적절한 내용 하나를 선택하여 이미 EOSIO 기반 블록체인에 배포된 다중 인덱스 테이블의 구조를 변경할 수 있습니다.

#### 1. 기존 데이터를 잃어도 상관 없다면

기존 데이터가 전부 삭제되어도 상관 없다면 다음 과 같이 진행합니다.

1. 첫 번째 테이블에서 모든 레코드를 삭제한다.
2. 수정된 테이블 구조와 새 컨트랙트를 배포한다.

#### 2. 기존 데이터를 유지하려면

기존 데이터를 유지하는 방법은 다음 두 가지가 있습니다.

**2.1 바이너리 확장(Binary Extension)을 사용**

바이너리 확장으로 구조를 변경하려면 이 자습서 내용을 참조합니다.

[https://developers.eos.io/manuals/eosio.cdt/latest/tutorials/binary-extension](https://developers.eos.io/manuals/eosio.cdt/latest/tutorials/binary-extension)

**2.2 ABI 변형 (ABI variants)를 사용한다.**

ABI 변형을 사용하여 구조를 변경하려면 이 자습서 내용을 참조합니다.

[https://developers.eos.io/manuals/eosio.cdt/latest/tutorials/abi-variants](https://developers.eos.io/manuals/eosio.cdt/latest/tutorials/abi-variants)

**2.3 두 번째 테이블로 기존 데이터를 마이그레이션 하려면**

2.3.1 느리지만 다운타임 없이 마이그레이션

1. 기존 테이블을 놔 두고 새 버전의 다중 인덱스 테이블을 만듭니다.
2. 이전 테이블에서 새 테이블로 데이터를 전송합니다. 일반적인 액세스 패턴의 일부로 이러한 작업을 수행할 수 있습니다. 먼저 새 테이블을 확인하여 원하는 항목이 있는지 확인하고 없으면 원래 테이블을 확인합니다. 원래 테이블에 데이터가 있다면 새 필드의 데이터를 추가하는 동안 마이그레이션한 다음 원래 테이블에서 제거하여 RAM 비용을 절감할 수 있습니다.
3. 마이그레이션을 완료할 때까지 두 버전의 다중 인덱스 테이블을 모두 유지해야 합니다. 이후 컨트랙트을 업데이트하여 다중 인덱스 테이블의 원래 버전을 제거할 수 있습니다.

2.3.2. 빠르지만 다운타임 발생. 코드 복잡성을 줄이고 애플리케이션의 다운타임을 허용할 수 있는 경우 다음과 같이 진행합니다.

1. 마이그레이션 목적으로만 새 버전의 계약을 배포하고 완료될 때까지 테이블의 모든 레코드에서 마이그레이션 트랜잭션을 실행합니다. 첫 번째 테이블이 큰 경우(예: 레코드 수가 많은 경우), 마이그레이션 트랜잭션을 실행하는 동안 트랜잭션 시간 제한에 도달할 수도 있습니다. 이를 피하려면 마이그레이션 기능을 구현하여 실행할 때마다 제한된 수의 레코드만을 이동시킵니다.
2. 마이그레이션 및 다운타임이 끝나면 새 버전의 테이블만 사용하는 새 컨트랙트를 배포합니다.

{% hint style="warning" %}
위의 두 가지 마이그레이션 방법 모두 어떤 식으로든 사전 계획(예: 사용자 피드백을 위해 컨트랙트를 유지 보수 모드로 전환하는 기능 등)이 필요합니다.
{% endhint %}

## 컨트랙트 보안

### 기본 추천사항

아래 내용은 스마트 컨트랙트의 보안을 위한 기본적인 권고 사항입니다.

### 1. 실행 권한 체크

아래 메소드 들은 EOSIO 라이브러리에 있으며, 스마트 컨트랙트의 실행 권한 체크 기능을 구현하는데 사용할 수 있습니다.

* has\_auth
* require\_auth
* require\_auth2
* require\_recipient

### 2. 자원 관리

작성한 컨트랙트가 RAM, CPU, NET 소비량에 얼마나 영향을 주는지, 그리고 어느 계정이 이러한 자원 사용료를 지불하는지 이해해야 합니다.

### 3. 보안은 기본

제품 계획 및 개발 시작점부터 보안과 관련된 고려 사항을 포함하는, 견고하고 포괄적인 개발 프로세스를 갖추도록 합니다.

### 4. 지속적 통합 및 지속적 배포(CI/CD)

컨트랙트를 업데이트하고 블록체인에 배포할 때 마다 테스트를 수행합니다. 작업을 간단히 하기 위하여, 가능한 테스트를 자동화 하며, 주기적으로 유지보수하도록 합니다.

### 5. 보안 감사

독립적인 스마트 컨트랙트 감사를 수행합니다. 가능하면 두 개 이상의 조직으로 부터 받는 것이 좋습니다.

### 6. 버그 헌팅

주기적으로 스마트 컨트랙트에 대한 버그 헌팅 대회를 개최합니다. 그리고 진짜 보안 결점으로 보고된 부분에 대하여는 언제든지 지속적으로 보상을 지급하도록 합니다.

## 에러 핸들링

스마트 컨트랙트는은 문자열 에러 메시지 대신 uint64\_t 에러 코드를 에러 상태를 표현하는 데 사용합니다. 그러나 EOSIO 와 EOSIO.CDT 는 uint64\_t 의 일부 영역을 특정한 목적을 위하여 예약해 두었습니다. 스마트 계약 개발자는 아래의 범위와 제한에 대하여 파악하고 있어야 합니다.

1. 0 - 4,999,999,999,999,999,9990: 스마트 컨트랙트 개발자가 컨트랙트 관련 오류 코드를 할당하는 데 사용할할 수 있습니다.
2. 5,000,000,000,000,000,000 - 7,999,999,999,999,999,999: EOSIO.CDT 컴파일러가 사용하기 위해 예약된 영역입니다. EOSIO.CDT 가 WASM 코드를 생성하더라도 컴파일러는 이 범위 내에서 자동으로 생성된 오류 코드 값을 사용할 수도 있습니다. 이 범위 내의 오류 코드는 특정한 방식으로 컴파일된 컨트랙트(일반적으로 오류의 의미는 작성한 ABI 파일의 오류 코드 값과 문자열을 매핑하여 전달됨)에 따라 다릅니다.
3. 8,000,000,000,000,000,000 - 9,999,999,999,999,999,999: EOSIO.CDT 컴파일러가 사용하기 위해 예약된 영역입니다. 이 범위의 오류 코드는 컨트랙트에 한정되지 않고 EOSIO.CDT 에서 생성된 코드와 관련된 일반적인 런타임 오류 조건을 전달하는 데 사용됩니다.
4. 10,000,000,000,000,000,000 - 18,446,744,073,709,551,615: EOSIO가 시스템 수준의 오류를 나타내기 위해 예약된 영역입니다. EOSIO 는 실제로 eosio\_asset\_code 가 실패하면 이 범위 내에 있는 에러 코드값만 사용하도록 제한됩합니다.

따라서 스마트 컨트랙트 개발자는 위 목록 중 첫 번째의 오류 코드 범위만을 자신의 컨트랙트에 사용해야 합니다.

## ABI/코드 생성자(Code Generator) 속성들

ABI 생성자 도구는 액션과 테이블을 표시하기 위하여 C++11 또는 GNU 스타일 속성을 사용합니다.

### \[\[eosio::action]]

이 속성은 메소드를 액션으로 표시하는데 사용합니다. 다음은 ABI 생성을 위하여 액션을 선언하는 4가지 방법입니다.

```cpp
// 아래는 C++11 이상의 버전에서 사용하는 속성 스타일입니다.
[[eosio::action]]
void testa( name n ) {
   // do something
}

// 아래는 GNU 스타일 속성으로, C 나 C++11 이하에서 사용할 수 있습니다.
__attribute__((eosio_action))
void testa( name n ){
   // do something
}

struct [[eosio::action]] testa {
   name n;
   EOSLIB_SERIALIZE( testa, (n) )
};

struct __attribute__((eosio_action)) testa {
   name n;
   EOSLIB_SERIALIZE( testa, (n) )
};
```

### \[\[eosio::table]]

아래와 같이 두 가지 방식으로 테이블의 ABI 를 생성할 수 있습니다.

```cpp
struct [[eosio::table]] testtable {
   uint64_t owner;
   /* all other fields */
};

struct __attribute__((eosio_table)) testtable {
   uint64_t owner;
   /* all other fields */
};

typedef eosio::multi_index<"tablename"_n, testtable> testtable_t;
```

만약 다중 인덱스를 사용하고 싶지 않다면, 다음과 같이 속성 안에 있는 이름을 명시적으로 지정하면 됩니다.

c++ \[\[eosio::table("\<valid action name>")]]

### \[\[eosio::contract("ANY\_NAME\_YOU\_LIKE")]]

```cpp
class [[eosio::contract("ANY_NAME_YOU_LIKE")]] test_contract : public eosio::contract {
};
```

위의 코드는 속성을 지정하여 이 클래스가 EOSIO 스마트 컨트랙트 라는 것을 나타내고 있으며, 이를 통해 컨트랙트의 네임스페이스를 사용할 수 있습니다. 예를 들어 eosio::token 이라는 헤더를 포함할 수 있고 eosio::token 의 액션과 테이블이 ABI 또는 만들어진 디스패처에 표시되지 않습니다.

### \[\[eosio::on\_notify("VALID\_EOSIO\_ACCOUNT\_NAME::VALID\_EOSIO\_ACTION\_NAME")]]

```cpp
[[eosio::on_notify("eosio.token::transfer")]]
void on_token_transfer(name from, name to, assert quantity, std::string memo) {
   // eosio.token 컨트랙트의 어떠한 계정에서 해당 컨트랙트가 배포된 계정으로 전송(transfer) 하는 액션 상에서 해야하는 일을 기술한다.
}

[[eosio::on_notify("*::transfer")]]
void on_any_transfer(name from, name to, assert quantity, std::string memo) {
   // 임의의 계약의 어떠한 계정에서 해당 컨트랙트가 배포된 계정으로 전송(transfer) 하는 액션 상에서 해야하는 일을 기술한다.
```

### \[\[eosio::wasm\_entry]]

```cpp
[[eosio::wasm_entry]]
void some_function(...) {
   // do something
}
```

위 코드는 어떤 함수를 진입점(entry point)으로 나타내는 것입니다. 그리고 해당 함수를 전역 생성자 또는 소멸자로 래핑합니다. 이로서는 eosio.cdt 툴체인을 사용하여 다른 에코시스템을 위한 WASM 바이너리를 만들 수 있습니다.

### \[\[eosio::wasm\_import]]

```cpp
extern "C" {
   __attribute__((eosio_wasm_import))
   void some_intrinsic(...);
}
```

위 코드는 웹 어셈블리를 가져오는 함수 선언을 나타냅니다. 이는 중복된 선언을 가진 보조 파일을 관리할 필요 없이 어떤 함수들이 가져오기 전용(예를 들어, "연결하지 않음") 함수들인지 지정하는 컴파일 모드를 사용할 수 있습니다.

### \[\[eosio::action, eosio::read-only]] <a href="#eosioaction-eosioread-only" id="eosioaction-eosioread-only"></a>

`read-only` 속성은 이 메소드가 읽기 전용 액션으로 마킹되었다는 것을 나타냅니다.

예제:

```cpp
[[eosio::action, eosio::read-only]]
std::vector<my_struct> get() {
   std::vector<my_struct> ret;
   // retrieve blockchain state and populate the ret vector
   return ret; 
}
```

액션이 읽기 전용으로 태깅되면 다음과 같은 특성을 가집니다.

* `Multi-Index API` 나 `Key Value API`의 insert/update 또는 write 함수를 호출 할 수 없습니다.
* `deferred transactions`을 호출할 수 없습니다.
* `inline actions`을 호출할 수 없습니다.
* 읽기 전용 질의가 반환하는 데이터 크기는 액션이 반환하는 값의 크기로 제한됩니다. 기본적으로 디폴트값(`default_max_action_return_value_size`,256 바이트)으로 설정됩니다.

읽기 전용으로 태깅된 액션이 insert/update/write 함수를 호출하거나 `deferred transactions` 나 `inline actions` 을 시도하면 `cdt-cpp` 와 `eosio-cc` 도구는 에러를 생성하고 컴파일을 종료시킵니다. 하지만 명령줄에서 오버라이드 옵션인 `--warn-action-read-only` 가 사용되면, `cdt-cpp` 와 `eosio-cc` 툴은 경고만 내보내고 컴파일을 계속합니다.

## 직접 ABI 파일 작성 및 수정하기

* 최신 버전의 ABI 에 포함된 고급 기능을 사용하려면 ABI 를 직접 작성해야 하며, 예외적이거나 고급 C++ 패턴이 사용될 경우 ABI 제너레이터의 타입 추론이 엉망이 될 수 있습니다. 따라서 ABI 작성 방법에 대해 잘 아는 것은 스마트 컨트랙트 작성자의 필수 지식입니다.
* ABI 의 다양한 섹션에 대해 알아보려면 ABI 파일 만들기를 참조합니다.다.[https://developers.eos.io/manuals/eosio.cdt/latest/best-practices/abi/understanding-abi-files#create-an-abi-file](https://developers.eos.io/manuals/eosio.cdt/latest/best-practices/abi/understanding-abi-files#create-an-abi-file)

### Ricardian 컨트랙트 및 구절을 ABI에 추가

* ABI 생성기는 ABI 생성 시 리카르디안 컨트랙트 및 구절을 자동으로 가져오려 할 것입니다. 여기에는 몇 가지 주의사항이 있는데, 하나는 각 Ricardian 컨트랙트와 구절을 표시하는 데 사용되는 HTML 태그와 파일의 엄격한 네이밍 지정 정책입니다.
* Ricardian 컨트랙트는은 \<contract name>.[contracts.md](http://name.contracts.md) 라는 이름의 파일에 저장되어야 하며, Ricardian 구절은 \<contract name>.clauses.md라는 파일에 저장되어야 합니다.
* 각 Ricardian 컨트랙트 파일에는 헤더 \<h1 class="contract"> ActionName \</h1> 가 있어야 합니다. ABI 생성기는 이 Ricardian 컨트랙트를을 지정된 액션에 가져다 붙이는 식으로 동작합니다.
* 각 Ricardian 구절에 대해 헤더 \<h1 class="clause"> ClauseID \</h1> 가 있어야 한다. 이렇게 하면 ABI 생성기가 구절 ID를 본문과 연결시킵니다.
* \-R 옵션을 cdt-cpp 및 eosio-abigen 에 추가하면 검색할 "리소스" 경로를 추가할 수 있습니다. 이렇게 하여 원하는 디렉토리에 파일을 배치하고 경로를 포함시키는 -I 옵션과와 같은 맥락으로 -R\<path to file>을 사용할 수 있습니다.
* hello.contracts.md 를 예제로 참조할 수 있습니다. [https://github.com/EOSIO/eosio.cdt/blob/master/examples/hello/ricardian/hello.contracts.md](https://github.com/EOSIO/eosio.cdt/blob/master/examples/hello/ricardian/hello.contracts.md)

## 스마트 컨트랙트 디버깅

스마트 컨트랙트를 디버그하려면 먼저 로컬 nodeos 노드를 만들어야 합니다. 로컬 nodeos 노드란 개인이 소유한 머신 위에서 동작하는 사설 테스트넷 또는 공개되어 있는 테스트넷에 연결된 노드를 의미합니다. 또한 이 로컬 노드는 contracts-console 옵션을 켠 상태에서 실행되어야 합니다. 즉, config.ini 파일에서 --contract-contracts= true 로 설정하거나, 명령줄에서 nodeos 를 실행할 때 --contracts-console 를 추가하여 실행중인 nodeos 가 로깅을 하도록 설정합니다. 그리고 디버그 시 출력된 로그를 확인합니다. 로깅에 대한 자세한 내용은 아래를 참조합니다.

스마트 컨트랙트를 처음 만드는 경우, 먼저 개인 테스트넷에서 스마트 컨트랙트를 테스트하고 디버깅 해 보면 전체 블록체인을 완벽하게 제어할 수 있고 적합한 로깅을 쉽게 추가할 수 있습니다. 또한 필요한 토큰을 무제한으로 조달 할 수 있으며 원할 때마다 블록체인의 상태를 재설정 할 수 있습니다. 프로덕션 준비가 되면 로컬 nodeos를 퍼블릭 테스트넷에 연결하여 테스트넷 로그를 로컬 노드에서 볼 수 있다.

본문에서는 개인의 사설 테스트넷 상에서의 디버깅에 대해 설명할 것입니다. 디버깅 방법 자체는 어디에서 해도 동일합니다.&#x20;

로컬 노드를 아직 설정하지 않은 경우 설치 안내서를 확인합니다. 기본적으로 퍼블릭 테스트넷에 nodeos 에 연결하도록 config.ini 파일을 설정하지 않았다면 로컬 노드는 개인 머신 위에서만 실행됩니다.

## 방법

스마트 컨트랙트 디버깅에 사용하는 주된 방법은 원시인 디버깅 입니다. Print API를 사용하여 스마트 컨트랙트에서 프린팅을 할 수 있으며 변수의 값과 계약의 흐름을 조사하는데 사용됩니다. Print C++ API는 C API를 래핑한 것이며, 일반적으로 추천하는 API 입니다.

{% hint style="info" %}
원시인 디버깅 (Caveman Debugging)이란 디버거를 사용하지 않고, print문같은 단순한 출력문 등으로 디버그를 위한 정보를 알아내는 기법입니다.
{% endhint %}

## Print

Print C API 는 다음과 같은 데이터 타입을 지원합니다.

* prints - null 로 종료하는 char 배열 (string)
* prints\_l - 주어진 크기가 있는 char 배열 (string)
* printi - 64-bit 부호 있는 정수.
* printui - 64-bit 부호 없는 정수.
* printi128 - 128-bit 부호 있는 정수.
* printui128 - 128-bit 부호 없는 정수.
* printsf - single-precision floating point number
* printdf - double encoded 된 64-bit의 부호 없는 정수
* printqf - quadruple encoded 된 64-bit 의 부호 없는 정수
* printn - base32 으로 인코딩 된 64 bit 이름 문자열
* printhex - 16 진수 데이터 바이너리와 그 크기

Print C++ API 는 print() 함수를 오버라이딩 하여 위의 C API 중 일부를 래핑하고 있습니다. 따라서 사용자는 어떤 print 함수를 사용해야 할 지 고민할 필요가 없습니다.

Print C++ API 는 다음과 같은 내용을 지원합니다.

* null 로 종료하는 char 배열, 즉 문자열(string).
* 부호 없는 128-bit, 부호 없는 64-bit, 부호 없는 32-bit, 부호 있는 정수, 부호 없는 정수
* 부호 없는 64-bit 정수로 인코딩 된 base32 문자열
* print() 메소드를 가지는 구조체

## 예시

아래는 스마트 컨트랙트를 디버깅 예시입니다.

#### debug.hpp

```cpp
namespace debug {
    struct foo {
        account_name from;
        account_name to;
        uint64_t amount;
        void print() const {
            eosio::print("Foo from ", eosio::name(from), " to ", eosio::name(to), " with amount ", amount, "\\n");
        }
    };
}
```

#### debug.cpp

```cpp
#include <debug.hpp>

extern "C" {

    void apply( uint64_t code, uint64_t action ) {
        if (code == N(debug)) {
            eosio::print("Code is debug\\n");
            if (action == N(foo)) {
                 eosio::print("Action is foo\\n");
                debug::foo f = eosio::unpack_action_data<debug::foo>();
               if (f.amount >= 100) {
                    eosio::print("Amount is larger or equal than 100\\n");
                } else {
                    eosio::print("Amount is smaller than 100\\n");
                    eosio::print("Increase amount by 10\\n");
                    f.amount += 10;
                    eosio::print(f);
                }
            }
        }
    }
} // extern "C"
```

#### debug.abi

```cpp
{
  "structs": [{
      "name": "foo",
      "base": "",
      "fields": {
        "from": "account_name",
        "to": "account_name",
        "amount": "uint64"
      }
    }
  ],
  "actions": [{
      "action_name": "foo",
      "type": "foo"
    }
  ]
}
```

이 컨트랙트를 배포하고 액션을 push 해 보겠습니다. debug 계정은 이미 만들어져 있으며 키도 지갑에 있다고 가정하겠습니다.

```cpp
$ cdt-cpp -abigen debug.cpp -o debug.wasm
$ cleos set contract debug CONTRACT_DIR/debug -p youraccount@active
$ cleos push action debug foo '{"from":"inita", "to":"initb", "amount":10}' --scope debug
```

로컬 nodeos 노드 로그를 보면 위의 메시지가 전송된 다음에 아래 내용이 표시되는 것을 볼 수 있습니다.

```cpp
Code is debug
Action is foo
Amount is smaller than 100
Increase amount by 10
Foo from inita to initb with amount 20
```

메시지가 올바른 흐름대로 가고 있고, amount 도 올바르게 업데이트 되었음을 알 수 있습니다.

아마도 위 메시지를 적어도 2번은 보게 될 것인데, 이는 각 트랜잭션이 검증(verification)과정, 블록 생성, 블록 어플리케이션에 각각 적용되기 때문입니다.

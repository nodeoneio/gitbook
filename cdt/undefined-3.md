# 액션으로부터 값을 반환하는 방법

## 개요

이 단원에서는 스마트 컨트랙트의 액션으로부터 값을 반환하는 방법을 다룹니다. 액션에서 값을 반환하려면 `return` 구문을 사용하며, 이 때 C++ 의 원시 타입, 표준 라이브러리 타입, 사용자 정의 타입을 값으로 반환할 수 있습니다.

## 시작하기 전에

다음과 같이 준비 되었는지 확인합니다.

* EOSIO 개발 환경.&#x20;
* 에러 없이 빌드되는 `smrtcontract` 라는 이름의 스마트 컨트랙트.
* `checkwithrv` 라는 이름의 액션. 이 액션으로부터 `action_response` 라는 사용자 정의 타입을 반환합니다.

다음과 같은 코드를 준비합니다.

```
struct action_response
{
  uint16_t id;
  std::pair<int, std::string> status;
};

class [[eosio::contract]] smrtcontract : public contract {
  public:
     using contract::contract;

     [[eosio::action]]
     action_response checkwithrv( name nm );
};
```

## 순서

다음과 같은 순서로 진행하면 `action_response` 라는 인스턴스를 반환하는 `checkwithrv` 액션을 만들 수 있습니다.

1. `action_response` 라는 C++ 사용자 정의 구조체의 인스턴스를 만듭니다.
2. 액션의 비즈니스 로직에 맞추어 데이터 멤버들을 초기화 합니다.
3. 초기화 및 인스턴스화 된 `action_response` 를 `return` 구문을 사용하여 넘깁니다.

```
[[eosio::action]]
action_response smrtcontract::checkwithrv( name nm ) {
  print_f("Name : %\n", nm);

  // create instance of the action response structure
  action_response results;

  // initialize its data members
  results.id = 1;
  if (nm == "hello"_n) {
     results.status = {0, "Validation has passed."};
  }
  else {
     results.status = {1, "Input param `name` not equal to `hello`."};
  }
  
  // use return statement
  return results; // the `set_action_return_value` intrinsic is invoked automatically here
}
```

반환값을 받는 액션을 만드는 다른 예제를 여기서 확인할 수 있습니다.\
[https://github.com/EOSIO/eosio.cdt/blob/develop/examples/hello/src/hello.cpp#L16](https://github.com/EOSIO/eosio.cdt/blob/develop/examples/hello/src/hello.cpp#L16)

## 다음 단계

* 작성한 스마트 컨트랙트를 컴파일하고 EOSIO 테스트넷 등의 블록체인 네트워크에 배포합니다.
* cleos 명령을 사용하여 `checkwithrv` 액션을 스마트 컨트랙트에 전달한 뒤 cleos 콘솔 출력에서 반환값을 확인합니다.
* 다른 프로그래밍 수단을 사용하여 `checkwithrv` 액션을 스마트 컨트랙트에 전달한 뒤 액션 추적에서 반환값을 확인합니다.

{% hint style="info" %}
액션의 반환값은 RPC API 를 사용하여로 액션을 전송하는 클라이언트만 사용할 수 있습니다. 인라인 액션은 비동기적으로 실행되기 때문에 현재로서는 인라인 액션에 반환값을 도입할 계획은 없습니다.
{% endhint %}

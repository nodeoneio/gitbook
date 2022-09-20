# 스마트 컨트랙트에서 사용자 인증 방법

이 단원에서는 스마트 컨트랙트에서 사용자를 인증하는 방법을 알아보겠습니다.

## 시작하기 전에

1. hi 액션이 정의되고 구현된 스마트 컨트랙트 소스코드가 있어야 합니다.
2. hi 액션은 하나의 `name` 타입 `user` 를 파라미터로 받습니다.

## 코드 레퍼런스

다음 코드 레퍼런스 가이드에서 스마트 컨트랙트에서 사용자 인증을 구현하는데 사용할 수 있는 함수들을 확인할 수 있습니다.

* function [has\_auth(name n)](https://developers.eos.io/manuals/eosio.cdt/latest/namespaceeosio#function-has\_auth)
* function [require\_auth(name n)](https://developers.eos.io/manuals/eosio.cdt/latest/namespaceeosio/#function-require\_auth-12)
* function [require\_auth2(capi\_name name, capi\_name permission)](https://developers.eos.io/manuals/eosio.cdt/v1.8/group\_\_action\_\_c#function-require\_auth2)
* function [check(bool pred, ...)](https://developers.eos.io/manuals/eosio.cdt/latest/group\_\_system/#function-check)

## 순서

다음은 hi 액션에서 user 계정을 인증하는 방법에 대한 순서입니다. 스마트 컨트랙트 액션에서 사용자를 인증할 수 있는 방법은 3가지가 있습니다. 필요에 따라 아래 3가지 중 아무 메소드나 가져다 사용할 수 있습니다.

* [Use check(...) in combination with has\_auth(...)](https://developers.eos.io/manuals/eosio.cdt/latest/how-to-guides/authorization/how\_to\_restrict\_access\_to\_an\_action\_by\_user/#1-use-checkhas\_auth)
* [Use require\_auth(...)](https://developers.eos.io/manuals/eosio.cdt/latest/how-to-guides/authorization/how\_to\_restrict\_access\_to\_an\_action\_by\_user/#2-use-require\_auth)
* [Use require\_auth2(...)](https://developers.eos.io/manuals/eosio.cdt/latest/how-to-guides/authorization/how\_to\_restrict\_access\_to\_an\_action\_by\_user/#3-use-require\_auth2)

### has\_auth() 와 조합하여 사용하는 Use Check

다음은, 계정이 트랜잭션 서명에 사용하는 권한(예: `owner`, `active`, `code`)에 관계없이, 액션의 파라미터로 전송된 계정에 의해서만 액션 hi가 실행되도록 강제하는 코드입니다.

{% hint style="info" %}
보다 정확한 메시지를 전달하기 위해 커스텀 에러 메시지를 사용할 수 있습니다.
{% endhint %}

```
#include <capi/eosio/action.h>

void hi( name user ) {
   check(has_auth(user), "User is not authorized to perform this action.");
   print( "Hello, ", name{user} );
}
```

다른 예제를 [틱택토 튜토리얼](https://developers.eos.io/welcome/latest/tutorials/tic-tac-toe-game-contract/#action-handler---move) 에서 찾아볼 수 있습니다.

### require\_auth 사용

다음은, 계정이 트랜잭션 서명에 사용하는 권한(예: `owner`, `active`, `code`)에 관계없이 액션의 파라미터로 전송된 계정에 의해서만 액션 hi가 실행되도록 강제하는 코드입니다.

```
void hi( name user ) {
   require_auth( user );
   print( "Hello, ", name{user} );
}
```

{% hint style="info" %}
이 메소드는 일반적인 사용자 인증 오류 메시지가 반환하며, 커스텀 메시지를 사용할 수 없습니다.
{% endhint %}

### require\_auth2 사용

다음은, 계정이 트랜잭션 서명에 사용하는 권한이 `active`인 경우에만 액션의 파라미터로 전송된 계정에 의해서만 액션 hi 가 실행되도록 강제하는 코드입니다. 즉 사용자가 다른 권한(예: `owner`, `code)`을 사용하는 경우 액션을 실행할 수 없습니다.

```
#include <capi/eosio/action.h>

void hi( name user ) {
   require_auth2(user.value, "active"_n.value);
   print( "Hello, ", name{user} );
}
```

{% hint style="info" %}
이 메소드에서도 마찬가지로 커스텀 메시지를 사용할 수 없고 일반적인 사용자 인증 오류 메시지가 반환됩니다.
{% endhint %}

# Hello World Contract!

한번 블록체인에 저장된 각 트랜잭션 레코드는 수정하거나 삭제할 수 없습니다. 스마트 컨트랙트는 오직 블록체인 온체인 상에 상태(State)를 저장하고 업데이트하는 작업만 수행 합니다.&#x20;

스마트 컨트랙트는 C++ 클래스의 형식을 가지며, 스마트 컨트랙트의 동작을 정의하는 액션은 C++ 메소드 형식을 가집니다. 스마트 컨트랙트 액션은 블록체인 위에서 실행되고, 블록체인을 사용하는 클라이언트 애플리케이션은 이 액션을 호출하는 코드로 작성됩니다.

이제 역사와 전통을 자랑하는 "Hello World" 를 출력하는 간단한 스마트 컨트랙트를 만들어 보겠습니다.

본 단원에서는 다음과 같은 주요 개념을 알아볼 것입니다.

* CDT: 스마트 컨트랙트를 빌드하는데 사용되는 툴체인 및 라이브러리.
* 웹어셈블리(Web Assembly, WASM): nodeos 에서 호스트 되는 가상 머신(eos-vm)이 사용하는 실행가능한 바이너리 코드 형식.
* ABI(Application Binary Interface): 웹 어셈블리와 가상머신 간에 오가는 데이터를 어떻게 직렬화 할지 정의하는 인터페이스.
* 스마트 컨트랙트(Smart Contract): 블록체인상에서 실행되는 액션과 트랜잭션이 정의되어 있는 코드.&#x20;

## 이 단원에서 다루는 내용

* `hi` 액션을 가지고 있는 간단한 스마트 컨트랙트 작성법.
* 스마트 컨트랙트를 컴파일하고 Antelope 기반 블록체인에 배포하는 방법.
*  스마트 컨트랙트의 `hi` 액션을 호출하는 명령어 사용법.

## 시작하기 전에

* 기초적인 C++ 프로그램 코드를 이해할 수 있어야 합니다.
* 스마트 컨트랙트를 작성할 에디터나 IDE 가 필요합니다.
* Antelope 기반 로컬 블록체인 개발 환경이 준비되어 있어야 합니다.

## 컨트랙트 개발 툴킷(CDT)

Antelope 기반 블록 체인은 사용자가 작성한 스마트 컨트랙트를 실행할 때 WebAssembly(WASM) 형식을를 사용합니다.&#x20;

스마트 컨트랙트를 작성하고 컴파일 하는 데 필요한 라이브러리와 도구는 컨트랙트 개발 툴킷(Contract Development Toolkit), 줄여서 CDT 라 부르는 툴킷에서 제공하고 있습니다. 이 툴 체인은 clang/llvm C/C++ 컴파일러를 기반으로 하며, 이는 WASM 애플리케이션을 컴파일 하는데 있어 가장 발전된 도구 중 하나입니다. 이를 사용하여 Antelope 체인에서 실행되는 스마트 컨트랙트의 WASM 을 만들게 됩니다.

이제 [CDT 설치](../cdt-contract-development-toolkit/) 단원을 참조하여 최신 버전의 CDT 를 설치합니다.

Antelope WASM은 다양한 언어로 만들 수 있지만 성능과 편의성을 고려할 때 보편적으로 사용하는 언어는 C++ 입니다. 따라서, 너무 깊게 학습할 필요까지는 없지만, 모던 C++(C++11 이상) 의 기본적인 개념과 사용법은 학습하여 두실 필요가 있습니다.

스마트 컨트랙트를 블록체인에 배포하려면 먼저 `cdt-cpp` 를 사용하여 스마트 컨트랙트를 컴파일 해야 합니다. 스마트 컨트랙트 소스 파일을 컴파일하면 다음과 같은 파일이 만들어집니다.

* 웹 어셈블리 파일(`.wasm`)은 Antelope 블록체인의 웹 어셈블리 엔진이 실행하는 바이너리 코드입니다. 웹 어셈블리(wasm) 엔진은 `nodeos` 데몬에서 호스팅되고 있으며 스마트 컨트랙트 코드를 실행합니다.&#x20;
* 응용 프로그램 바이너리 인터페이스(ABI, Application Binary Interface) 파일(`.abi`)은 wasm 엔진으로 데이터가 오고 나갈갈 때 어떻게 직렬화/역직렬화 할 것인가를 정의합니다.

## 컨트랙트 만들기

이제 Hello World 스마트 컨트랙트를 만들어 봅시다. 일반적으로 스마트 컨트랙트를 만들 때는 클래스 선언이 들어 있는 헤더 파일(`.hpp`)과 스마트 컨트랙트 액션이 구현되어 있는 C++ 소스 파일(`.cpp`) 두 가지 파일을 만들게 됩니다. 본 예제는 매우 간단하기 때문에 `.cpp` 소스 파일만 작성하겠습니다.

### hello.cpp 파일 만들기

먼저 작업할 워킹 디렉토리를 하나 지정합니다. 본 예제에서는 `~/workspace` 를 사용할 것입니다. 그 아래에 **hello** 라는 이름을 가진 디렉토리를 하나 만들고 들어갑니다. 여기에서 스마트 컨트랙트 소스 파일을 작성하겠습니다.

```shell
cd ~/workspace
mkdir hello
cd hello
```

`hello.cpp` 라는 이름의 파일을 하나 만들고 에디터로 열어서 편집할 준비를 합니다.

```shell
touch hello.cpp
```

이제 코드를 작성하겠습니다. 다음과 같은 네 단계를 따라 `hello.cpp` 파일에  코드를 작성합니다.

a. 다음과 같이 `eosio` 베이스 라이브러리를 `include` 합니다. `eosio.hpp` 에는 `eosio::contract` 와 같이 스마트 컨트랙트를 작성할 때 필요한 클래스들이 포함되어 있습니다.

```cpp
#include <eosio/eosio.hpp>
```

b. 코드를 간결하게 작성하기 위해 using 으로 eosio 네임스페이스를 선언합니다. 예를 들어 네임스페이스를 사용하면 `eosio::print("roy")`대신 `print("roy")`와 같이 작성할 수 있습니다.

```cpp
using namespace eosio;
```

b. `eosio::contract` 를 상속하는 표준 C++ 11 클래스 hello 를 만듭니다. 그리고 클래스에`[[eosio::contract]]` 속성을 지정하면 CDT 컴파일러가 이것이 스마트 컨트랙트라는 것을 알게 됩니다.

```cpp
class [[eosio::contract]] hello : public eosio::contract {};
```

다른 방법으로 다음과 같이 `class [[eosio::contract]]` 부분을 `CONTRACT` 매크로로 치환하여 표기하는 방법도 있습니다.

```cpp
CONTRACT hello : public contract { 
```

이 쪽이 보다 간결하기 때문에 매크로를 이용하는것도 좋은 방법입니다.

`EOSIO.CDT` 컴파일러는 자동으로 메인 디스패처(main dispatcher) 와 `ABI 파일`을 만듭니다.  [디스패처](https://developers.eos.io/manuals/eosio.cdt/latest/group\_\_dispatcher/?query=dispatcher\&page=1#dispatcher)는 액션 호출을 루팅(route)하여 올바른 스마트 컨트랙트 액션이 호출되도록 합니다. 컴파일러는  `eosio::contract` 속성을 사용할 때 하나의 디스패처를 생성합니다. 숙련된 개발자인 경우는 스스로 만든 디스패처를 사용하여 이러한 동작을 커스터마이징 할 수 있습니다.

c. `Public` 접근지정자와 `using` 선언문을 추가하여 `eosio::contract`로 부터 베이스 클래스 멤버들을 가져옵니다. 이제 디폴트 베이스 클래스 생성자를 사용할 수 있습니다.

```cpp
public:
	using contract::contract;
```

d. 이제 퍼블릭 액션 `hi` 를 추가해 봅시다. Antelope 는 독자적으로 정의된 여러가지 타입을 가지고 있는데, 그중에서도 `name` 은 가장 많이 사용되는 타입 중 하나입니다. 이 액션은 `eosio::name` 타입의 매개변수를 받으며, `eosio::print()` 함수를 사용하여 "**Hello World, "** 문자열과 `eosio::name` 타입의 매개변수로 받은 `user` 를 연결하여 출력합니다.

```cpp
	[[eosio::action]] 
	void hi( name user ) {
		print( "Hello World, ", user);
	}
```

`[[eosio::action]]` 은 이것이 액션임을 컴파일러에게 알려주는 속성입니다. 이 또한 다음과 같이 `ACTION` 매크로를 사용하여 간결하게 표시할 수 있습니다.

```cpp
ACTION hi(name user) {
   print("Hello World, ", user);
}
```

지금까지 작성한 `hello.cpp` 파일은 다음과 같습니다.

```cpp
#include <eosio/eosio.hpp>

using namespace eosio;

CONTRACT hello : public contract {
  public:
      using contract::contract;
      ACTION void hi( name user ) {
         print( "Hello World, ", user);
      }
};
```

`eosio::print` 함수는 `eosio/eosio.hpp` 에 선언되어 있습니다. 스마트 컨트랙트는 이 함수를 사용하여 Hello 문자열과 매개변수로 받은 `eosio::name` 타입의 `user` 를 연결하여 출력합니다.

이제 파일을 저장합니다.

## 컴파일 및 배포

간단한 스마트 컨트랙트 코드를 작성하였습니다.  이제 컴파일 하고 블록체인에 배포해 보겠습니다. 웹 어셈블리(`.wasm`) 파일과 `abi` 파일을 만드려면 CDT 툴킷의 `cdt-cpp` 명령을 사용합니다.

### 컴파일 및 배포 순서

다음과 같이의`cdt-cpp` 명령으로 `hello.cpp`  파일을 컴파일 합니다. 명령은 `hello.cpp`파일이 들어있는 디렉토리에서 실행합니다. 아니면 절대경로 또는 상대경로와 파일명을 조합하여 사용할 수도 있습니다.

```shell
cdt-cpp -abigen -o hello.wasm hello.cpp
```

위 명령을 실행하면 `hello.wasm` 과 `hello.abi` 라는 두 개의 파일이 만들어집니다. 리카르디안 구절(clause)과 컨트랙트가 없다는 경고가 출력되지만 무시해도 문제 없습니다. 리카르디안 구절과 컨트랙트는 별도의 단원에서 다룰 것입니다.

이제 `hello.wasm` 과 `hello.abi` 파일을 블록체인상의 `hello` 라는 계정으로 배포하겠습니다. 아직 `hello` 계정이 없다면 [계정과 권한](../../basic-antelope-leap/basic-accounts-and-permissions.md) 단원을 참조하시기 바랍니다. `hello.wasm` 과 `hello.abi` 가 들어있는 디렉토리 외부에서 명령을 실행하는 경우 `contract-dir` 옵션을 추가하여 `.wasm` 과 `.abi` 파일이 들어있는 위치를 지정할 수 있습니다.

```shell
# cleos set contract [컨트랙트 이름] [경로] -p [계정@권한]
cleos set contract hello . -p hello@active
```

{% hint style="info" %}
위 명령을 실행할 때 지갑이 잠금 해제(unlock) 되어 있는지 확인합니다. `cleos` 명령이 실행될 때 지갑에 저장된 개인키로 트랜잭션을 서명할 권한을 얻어야 합니다. `cleos` 는 자동으로 잠금 해제되어 열린 지갑에서 권한에 필요한 개인 키를 찾습니다. 위 명령의 경우 `-p hello@active` 권한이 될 것입니다. 상세한 내용은 `cleos set contract --help` 명령으로 도움말에서 확인할 수 있습니다.
{% endhint %}

### 스마트 컨트랙트 액션 호출하기 <a href="#calling-a-smart-contract-action" id="calling-a-smart-contract-action"></a>

스마트 컨트랙트가 성공적으로 배포되고 나면 스마트 컨트랙트 액션을 블록체인에 푸시하여 `hi` 액션을 테스트할 수 있습니다. 여기서 alice, bob 이라는 계정을 활용할 것입니다. 이 계정들이 없다면 [계정과 권한](../../basic-antelope-leap/basic-accounts-and-permissions.md) 단원을 참조하여 만들어 놓습니다.

#### Hi 액션을 호출하는 순서 <a href="#procedure-to-call-the-hi-action" id="procedure-to-call-the-hi-action"></a>

다음과 같이 bob 계정과 권한으로 `cleos push action` 명령을 사용합니다.

```shell
$ cleos push action hello hi '["bob"]' -p bob@active
executed transaction: 4c10c1426c16b1656e802f3302677594731b380b18a44851d38e8b5275072857  244 bytes  1000 cycles
#    hello.code <= hello.code::hi               {"user":"bob"}
>> Hello World, bob
```

아무 사용자나 `hello` 컨트랙트의 `hi` 액션을 호출할 수 있습니다. 다른 계정을 한번 사용해 보겠습니다.

```shell
$ cleos push action hello hi '["alice"]' -p alice@active
executed transaction: 28d92256c8ffd8b0255be324e4596b7c745f50f85722d0c4400471bc184b9a16  244 bytes  1000 cycles
#    hello.code <= hello.code::hi               {"user":"alice"}
>> Hello World, alice
```

위에서 만든 Hello World 스마트 컨트랙트는 매우 간단한 것이며 `hi` 액션을 아무나 호출할 수 있습니다. 실제 사용하는 스마트 컨트랙트는 보안이 중요하기 때문에 인증을 위한 코드를 더 추가해 보겠습니다.&#x20;

## 인증(Authorisation)

Antelope 블록체인은 비대칭 암호화 기술을 사용하여 트랜잭션을 푸시하는 계정이 그 계정의 개인 키로 트랜잭션에 서명했는지 확인할 수 있습합니다. 블록체인은 계정 권한 테이블을 사용하여 계정에 작업을 수행하는 데 필요한 권한이 있는지 확인합니다.&#x20;

인증을 사용하는 것이 [스마트 컨트랙트의 보안](https://developers.eos.io/manuals/eosio.cdt/latest/best-practices/securing\_your\_contract)을 지키는 가장 기본적인 단계입니다. 인증 검사에 대한 자세한 내용은 [스마트 컨트랙트에서의 사용자 인증 방법](../../smart-contract-advanced/smart-contract-authorization.md) 단원에서 확인할 수 있습니다.

스마트 컨트랙트에 `require_auth` 를 추가하면 액션이 실행될 때 `require_auth` 함수가 권한이 부여된 사용자와 `name` 매개 변수가 일치하는지 확인합니다.

### 인증을 추가하는 순서

`hello.cpp` 를 다음과 같이 수정하여 `require_auth` 를 사용하도록 합니다.

```cpp
void hi( name user ) {
   require_auth( user );
   print( "Hello World, ", name{user} );
}
```

컨트랙트를 다시 컴파일 합니다.&#x20;

```shell
cdt-cpp -abigen -o hello.wasm hello.cpp
```

수정된 스마트 컨트랙트를 다시 배포합니다.

```shell
cleos set contract hello . -p hello@active
```

액션을 다시 호출해 봅니다. 이번에는 인증받지 못하는 계정을 사용해 볼 것입니다. 아래 명령은 alice 의 권한을 사용하여 트랜잭션에 서명하지만 계정 bob 을 hi 액션으로 전달합니다.

```shell
cleos push action hello hi '["bob"]' -p alice@active
```

`require_auth` 는 트랜잭션을 중단시키고 다음과 같이 에러 메시지를 출력 할 것입니다.

```
error 2022-09-14T17:03:24.566 cleos     main.cpp:575                  print_result         ] soft_except->to_detail_string(): 3090004 missing_auth_exception: Missing required authority
missing authority of bob
    {"account":"bob"}
    nodeos  apply_context.cpp:255 require_authorization
pending console output:
    {"console":""}
    nodeos  apply_context.cpp:123 exec_one
```

이제 이 컨트랙트는 `name` 파라미터와 트랜잭션에 서명한 계정이 동일한 계정인지 확인할 수 있게 되었습니다.

다시 해봅시다. 이번에도 alice 계정의 권한을 사용하지만 hi 에 넘기는 이름을 alice 로 할 것입니다.

```shell
cleos push action hello hi '["alice"]' -p alice@active
```

실행 결과는 다음과 같습니다.

```
235bd766c2097f4a698cfb948eb2e709532df8d18458b92c9c6aae74ed8e4518  244 bytes  1000 cycles
#    hello <= hello::hi               {"user":"alice"}
>> Hello World, alice
```

액션을 호출하는 계정과 액션이 `name` 으로 넘겨받은 계정이 같은 인증 계정인지 확인된 후에 액션이 성공적으로 실행되는 것을 볼 수 있습니다.

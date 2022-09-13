# Hello World Contract!

Leap 소프트웨어를 인스톨하여 개발 환경을 마련하였습니다. 이제 스마트 컨트랙트를 만들어 블록체인에 배포하고 실행해 보겠습니다.&#x20;

블록체인에 저장된 각 트랜잭션 레코드는 수정하거나 삭제할 수 없으며 스마트 컨트랙트는 블록체인에 상태를 저장하고 업데이트 합니다. 블록체인 애플리케이션은 스마트 컨트랙트 액션을 호출하는 클라이언트 코드로 구성되며, 스마트 컨트랙트 액션은 블록체인 위에서 실행됩니다.

먼저 역사와 전통을 자랑하는 "Hello World" 를 출력하는 간단한 스마트 컨트랙트부터 시작해 보겠습니다.

본 단원에서는 다음과 같은 주요 개념을 알아볼 것입니다.

* CDT: 스마트 컨트랙트를 빌드하는데 사용되는 툴체인 및 라이브러리
* 웹어셈블리(Web Assembly, WASM): nodeos 에서 호스트 되는 가상 머신(eos-vm)이 사용하는 실행가능한 바이너리 코드 형식
* ABI(Application Binary Interface): 웹 어셈블리와 가상머신 간에 데이터를 어떻게 직렬화 할지 정의하는 인터페이스.
* 스마트 컨트랙트(Smart Contract): 블록체인상에서 실행되는 액션과 트랜잭션이 정의되어 있는 코드.&#x20;

## 이 단원에서 다루는 내용

* `hi` 액션을 가지고 있는 간단한 스마트 컨트랙트.
* 스마트 컨트랙트를 컴파일하고 Antelope 기반 블록체인에 배포.
*  스마트 컨트랙트의 `hi` 액션을 호출하는 명령어 사용법.

## 시작하기 전에

* 기초적인 C++ 프로그래밍 언어를 이해할 수 있음.
* 스마트 컨트랙트를 작성할 에디터나 IDE 를 준비.
* Antelope 기반 로컬 블록체인 개발 환경 준비.

## 컨트랙트 개발 툴킷(CDT)

Antelope 기반 블록 체인은 WebAssembly(WASM)를 사용하여 사용자가 생성한 애플리케이션과 코드를 실행합니다.&#x20;

스마트 컨트랙트를 구축하는 데 필요한 라이브러리와 도구는 컨트랙트 개발 툴킷(Contract Development Toolkit), 줄여서 CDT 라 부르는 툴킷에서 제공하고 있습니다. 이 툴 체인은 clang/llvm C/C++ 컴파일러를 기반으로 하는데, 이는 WASM 애플리케이션을 컴파일 하는데 있어 가장 발전된 도구 중 하나입니다. 이를 사용하여 스마트 계약의 WASM 을 만들 것입니다. 자세한 내용은 [CDT 설명서](broken-reference)를 참조하시기 바랍니다.

Antelope WASM은 다양한 언어로 만들 수 있지만 성능과 편의성을 고려할 때 보편적으로 사용하는 언어는 C++ 입니다. 따라서, 너무 깊게 학습할 필요까지는 없지만, 모던 C++(C++11 이상) 의 기본적인 개념과 사용법은 학습하여 두실 필요가 있습니다.

스마트 컨트랙트를 블록체인에 배포하려면 먼저 `eosio-cpp` 를 사용하여 스마트 컨트랙트를 컴파일 해야 합니다. 스마트 컨트랙트 소스 파일을 컴파일하면 웹 어셈블리 파일과 응용 프로그램 바이너리 인터페이스(ABI, Application Binary Interface) 파일이 만들어집니다.

* 웹 어셈블리 파일(.wasm)은 Antelope 블록체인의 웹 어셈블리 엔진이 실행하는 바이너리 코드입니다. 웹 어셈블리(wasm) 엔진은 `nodeos` 데몬에서 호스팅되고 있으며 스마트 컨트랙트 코드를 실행합니다.&#x20;
* 응용 프로그램 바이너리 인터페이스 파일(.abi 파일)은 wasm 엔진으로 데이터가 오고 나갈갈 때 어떻게 직렬화/역직렬화 할 것인가를 정의합니다.

## 컨트랙트 만들기

다음 절차에 따라 Hello World 스마트 컨트랙트를 만들 수 있습니다. 일반적으로 스마트 컨트랙트를 만들 때는 클래스 선언이 들어 있는 헤더 파일(.hpp)과 스마트 컨트랙트 액션이 구현되어 있는 C++ 소스 파일(.cpp) 두 가지 파일을 만들게 됩니다. 본 예제는 간단하기 때문에 .cpp 파일만 사용하겠습니다..

### hello.cpp 파일 만들기

먼저 작업할 워킹 디렉토리를 하나 지정합니다. 본 예제에서는 `~/workspace` 를 사용할 것입니다. 그 아래에 **hello** 라는 이름을 가진 디렉토리를 하나 만들고 들어갑니다. 여기에 스마트 컨트랙트 파일을 저장하겠습니다.

```shell
cd ~/workspace
mkdir hello
cd hello
```

`hello.cpp` 라는 이름의 파일을 하나 만들고 에디터로 열어서 편집할 준비를 합니다.

```shell
touch hello.cpp
```

이제 스마트 컨트랙트 코드를 작성하겠습니다. 다음과 같은 네 단계를 따라 `hello.cpp` 파일에  코드를 작성합니다.

a. 다음과 같이 `eosio` 베이스 라이브러리를 include 합니다. `eosio.hpp` 에는 `eosio::contract` 와 같이 스마트 컨트랙트를 작성할 때 필요한 클래스들이 포함되어 있습니다.

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

`EOSIO.CDT` 컴파일러는 자동으로 메인 디스패처(main dispatcher) 와 `ABI 파일`을 만듭니다.  [디스패처](https://developers.eos.io/manuals/eosio.cdt/latest/group\_\_dispatcher/?query=dispatcher\&page=1#dispatcher)는 액션 호출을 루팅(route)하여 올바른 스마트 컨트랙트 액션이 호출되도록 합니다. 컴파일러는  `eosio::contract` 속성을 사용할 때 하나의 디스패처를 생성합니다. 숙련된 개발자는 스스로 정의한 디스패처를 사용하여 이러한 동작을 커스터마이징 할 수 있습니다.

c. Public 접근지정자와 함께 using 선언문을 추가하여 `eosio::contract`로 부터 베이스 클래스 멤버들을를 가져옵니다. 이제 디폴트 베이스 클래스 생성자를 사용할 수 있습니다.

```cpp
public:
	using contract::contract;
```

d. 이제 퍼블릭 액션 `hi` 를 추가해 봅시다. Antelope 는 여러가지 독자적인 타입 정의를 가지고 있는데, name 은 가장 많이 사용되는 타입 중 하나입니다. 이 액션은 `eosio::name` 타입의 매개변수를 받으며, `eosio::print()` 함수를 사용하여 "**Hello"** 문자열과 `eosio::name` 타입의 매개변수로 받은 user 를 연결한 뒤 출력합니다.

```cpp
	[[eosio::action]] 
	void hi( name user ) {
		print( "Hello, ", user);
	}
```

`[[eosio::action]]` 은 이것이 액션임을 컴파일러에게 알려주는 속성입니다. 이 또한 다음과 같이 `ACTION` 매크로를 사용하여 간결하게 표시할 수 있습니다.

```cpp
ACTION hi(name user) {
   print("Hello!, ", user);
}
```

\
지금까지 작성한 hello.cpp 파일은 다음과 같습니다.

```cpp
#include <eosio/eosio.hpp>

using namespace eosio;

CONTRACT hello : public contract {
  public:
      using contract::contract;
      ACTION void hi( name user ) {
         print( "Hello, ", user);
      }
};
```

`eosio::print` 함수는 `eosio/eosio.hpp` 에 선언되어 있습니다. 스마트 컨트랙트는 이 함수를 사용하여 Hello 문자열과 매개변수로 받은 `eosio::name` 타입의 user 를 연결하여 출력합니다.

이제 파일을 저장합니다.

## 컴파일 및 디플로이

스마트 컨트랙트 코드를 작성하었습니다.  이제부터 컴파일 하고 블록체인에 배포해 보겠습니다. 웹 어셈블리(.wasm) 파일과 abi 파일을 만드려면 CDT 툴킷의 `eosio-cpp` 명령을 사용합니다.

### 컴파일 및 디플로이 순서

다음의`eosio-cpp` 명령으로 `hello.cpp`  파일을 컴파일 합니다. 명령은 hello.cpp 파일이 들어있는 디렉토리에서 실행합니다. 아니면 절대경로 또는 상대경로와 파일명을 조합하여 사용할 수 도 있습니다.

```shell
eosio-cpp -abigen -o hello.wasm hello.cpp
```

위 명령을 실행하면 `hello.wasm` 과 `hello.abi` 라는 두 개의 파일이 만들어집니다.

이제 `hello.wasm` 과 `hello.abi` 파일을 블록체인상의 hello 라는 계정으로 배포하겠습니다. 아직 hello 계정이 없다면 [계정과 권한](https://developers.eos.io/welcome/latest/smart-contract-guides/before-you-begin/accounts-and-permissions) 단원을 참조하시기 바랍니다. Run the command outside the folder containing the `hello.wasm` 과 `hello.abi` 가 들어있는 디렉토리 외부에서 명령을 실행하는 경우 `contract-dir` 옵션을 추가하여 `.wasm` 과 `.abi` 파일이 들어있는 위치를 지정할 수 있습니다.

```shell
cleos set contract hello ./hello -p hello@active
```

{% hint style="info" %}
위 명령을 실행할 때 지갑이 잠금 해제(unlock) 되어 있는지 확인합니다. cleos 명령은 지갑에 저장된 개인키로 트랜잭션을 서명할 권한을 얻어야 합니다. cleos 는 자동으로 잠금 해제되어 열린 지갑에서 권한에 필요한 개인 키를 찾습니다. 위 명령의 경우 `-p hello@active` 권한이 될 것입니다. 상세한 내용은 `cleos set contract --help` 명령으로 도움말에서 확인할 수 있습니다.
{% endhint %}

### 스마트 컨트랙트 액션 호출하기 <a href="#calling-a-smart-contract-action" id="calling-a-smart-contract-action"></a>

일단 스마트 컨트랙트가 성공적으로 배포되고 나면 스마트 컨트랙트 액션을 블록체인에 푸시하고 `hi` 액션을 테스트할 수 있습니다

#### Hi 액션을 호출하는 순서 <a href="#procedure-to-call-the-hi-action" id="procedure-to-call-the-hi-action"></a>

다음과 같이 `cleos push action` 명령을 사용합니다.

```shell
$ cleos push action hello hi '["bob"]' -p bob@active
executed transaction: 4c10c1426c16b1656e802f3302677594731b380b18a44851d38e8b5275072857  244 bytes  1000 cycles
#    hello.code <= hello.code::hi               {"user":"bob"}
>> Hello, bob
```

이 컨트랙트는 아무 사용자에게나 hi 액션을 호출할 수 있게 허용합니다. 다른 계정을 한번 사용해 보겠습니다.

```shell
$ cleos push action hello hi '["alice"]' -p alice@active
executed transaction: 28d92256c8ffd8b0255be324e4596b7c745f50f85722d0c4400471bc184b9a16  244 bytes  1000 cycles
#    hello.code <= hello.code::hi               {"user":"alice"}
>> Hello, alice
```

위에서 만든 Hello World 스마트 컨트랙트는 매우 간단한 것이며 `hi` 액션을 아무나 호출할 수 있습니다. 실제 사용하는 스마트 컨트랙트는 보안이 중요하기 때문에 인증을 위한 코드를 더 추가해 보겠습니다. 이렇게 하면 스마트 컨트랙트가 어떤 계정이 이 액션을 호출하고 있는 것인지 확인합니다.

## 인증(Authorisation)

Mandel 블록체인은 비대칭 암호화를 사용하여 트랜잭션을 푸시하는 계정이 그 계정의 개인 키로 트랜잭션에 서명했는지 확인합니다. Mandel 블록체인은 계정 권한 테이블을 사용하여 계정에 작업을 수행하는 데 필요한 권한이 있는지 확인합니다. 인증을 사용하는 것이 [스마트 컨트랙트의 보안](https://developers.eos.io/manuals/eosio.cdt/latest/best-practices/securing\_your\_contract)을 지키는 첫 단계입니다. 인증 검사에 대한 자세한 내용을 보려면 이 [링크](https://developers.eos.io/manuals/eosio.cdt/latest/how-to-guides/authorization/how\_to\_restrict\_access\_to\_an\_action\_by\_user)를 클릭하십시오.

스마트 컨트랙트에 `require_auth` 를 추가하면 `require_auth` 함수가 액션을 실행하고 권한이 부여된 사용자와 name 매개 변수가 일치하는지 확인함으로서 부여된 권한을 확인합니다.

### 인증을 추가하는 순서

`hello.cpp` 를 다음과 같이 수정하여 `require_auth` 를 사용하도록 합니다.

```cpp
void hi( name user ) {
   require_auth( user );
   print( "Hello, ", name{user} );
}
```

컨트랙트를 다시 컴파일 합니다.&#x20;

```shell
eosio-cpp -abigen -o hello.wasm hello.cpp
```

수정된 스마트 컨트랙트를 다시 배포합니다.

```shell
cleos set contract hello ./hello -p hello@active
```

액션을 다시 호출해 봅니다. 이번에는 인증받지 못하는 계정을 사용해 볼 것입니다. 아래 명령은 alice 의 권한을 사용하여 트랜잭션에 서명하지만 계정 bob 을 hi 액션으로 전달합니다.

```shell
cleos push action hello hi '["bob"]' -p alice@active
```

`require_auth` 는 트랜잭션을 중단시키고 다음과 같이 출력 할 것입니다.

```
Error 3090004: Missing required authority
Ensure that you have the related authority inside your transaction!;
```

이제 이 컨트랙트는 name 파라미터와 트랜잭션에 서명한 계정이 동일한 계정인지 확인할 수 있게 되었습니다.

다시 해봅시다. 이번에도 alice 계정의 권한을 사용하지만 hi 에 넘기는 이름을 alice 로 할 것입니다.

```shell
cleos push action hello hi '["alice"]' -p alice@active
```

실행 결과는 다음과 같습니다.

```
235bd766c2097f4a698cfb948eb2e709532df8d18458b92c9c6aae74ed8e4518  244 bytes  1000 cycles
#    hello <= hello::hi               {"user":"alice"}
>> Hello, alice
```

액션을 호출하는 계정과 액션이 name 으로 넘겨받은 계정이 같은 인증 계정인지 확인된 후에 액션이 성공적으로 실행된 것을 볼 수 있습니다.

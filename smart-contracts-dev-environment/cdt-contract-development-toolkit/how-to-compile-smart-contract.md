# 스마트 컨트랙트 컴파일

## CLI 로 스마트 컨트랙트 컴파일

### 개요

본 단원에서는 명령줄 인터페이스에서 스마트 컨트랙트를 컴파일하는 방법에 대하여 설명할 것입니다. 컴파일 도구인 `cdt-cpp` 의 사용법은 다음과 같이 확인할 수 있습니다.

* 구 EOSIO 개발자 포털의 [cdt-cpp](https://developers.eos.io/manuals/eosio.cdt/v1.8/command-reference/eosio-cpp) 문서 확인.
* 명령줄에서 `cdt-cpp --help` 명령 실행.

### 시작하기 전에

* 로컬 폴더(예를 들어 `./examples/hello/`) 에 스마트 컨트랙트 소스가 들어 있어야 합니다. [Hello World Contract](../contract-dev-workflow/hello-world-contract.md) 단원에서 처음으로 스마트 컨트랙트를 만드는 방법에 대한 자세한 내용이 소개되어 있습니다.

### 순서

다음 순서대로 스마트 컨트랙트를 컴파일 합니다.

1. 위에서 예로 들었던 `./examples/hello` 폴더로 이동합니다. 그러면 `./src/hello.cpp` 파일을 볼 수 있습니다.
2.  다음 명령을 실행합니다.

    ```cpp
    mkdir build
    cd build
    cdt-cpp -abigen ../src/hello.cpp -o hello.wasm -I ../include/
    ```

    * `cdt-cpp`: `cdt-cpp` 도구.
    * `-abigen`: ABI 파일을 생성하기 위한 옵션.
    * `../src/hello.cpp`: 컴파일 할 cpp 소스 파일.
    * `-o hello.wasm`: `cdt-cpp` 가 만들어 낼 wasm 파일 이름.
    * `-I ../include/`: `cdt-cpp` 가 컴파일 시 포함해야 할 디렉토리 경로.
3. 다음 두개의 파일이 생성되었는지 확인합니다.
   * `hello.wasm`: 컴파일 된 wasm 바이너리.
   * `hello.abi`: 생성된 ABI 파일.

## Cmake 로 스마트 컨트랙트 컴파일하기

### CMake 구성하기

먼저 CMake 를 인스톨합니다. 자세한 내용은 [CMake installation page](https://cmake.org/install/) 를 확인합니다.

#### 순서

다음 두 가지 방식으로 CMake 를 구성할 수 있습니다.

#### 자동으로 CMake 구성을 생성

CMake 로 Antelope 스마트 컨트랙트를 컴파일 하려면 먼저 CMake 파일이 필요합니다. 다음과 같이 실행하여 `eosio-init` 툴로 `directory stub .hpp/.cpp` 와 CMake 구성 파일을 만듭니다.

```cpp
cd ~
eosio-init --path=. --project=test_contract
cd test_contract
cd build
cmake ..
make
ls -al test_contract
```

그러면 `~/test_contract/test_contract` 아래에 `test_contract.abi` 파일과 `test_contract.wasm` 파일이 생성됩니다. 이제 배포할 준비가 되었습니다.

#### 수동으로 CMake 를 구성

수동으로 CMake 를 구성하려면 example 폴더 안에 있는 `CMakeLists.txt` 템플릿 파일을 참조할 수 있습니다.

1.  CMakeList.txt

    ```cpp
    cmake_minimum_required(VERSION 3.5)
    project(test_example VERSION 1.0.0)

    find_package(eosio.cdt)

    add_contract( test test test.cpp )
    ```
2.  test.cpp

    ```cpp
    #include <eosio/eosio.hpp>
    using namespace eosio;

    class [[eosio::contract]] test : public eosio::contract {
    public:
       using contract::contract;

       [[eosio::action]] void testact( name test ) {
       }
    };

    EOSIO_DISPATCH( test, (testact) )
    ```
3. 다음과 같은 CMake 매크로가 제공됩니다.
   * `add_contract` 는 ABI 파일과 스마트 컨트랙트를 빌드하는데 사용됩니다. 첫 번째 매개변수는 컨트랙트의 이름이고 두 번째는 CMake 타겟 이름입니다. 그리고 나머지는 컨트랙트를 빌드하는 데 필요한 CPP 파일입니다.
   * `target_recar dian_directory` 는 특정 CMake 타겟에 리카르디안 컨트랙트가 있는 디렉토리를 추가하는데 사용합니다.
   * `add_native_library` 및 `add_native_executable` 은 네이티브 테스터를 위한 CMake 매크로로, `add_library` 및 `add_executable`을 대신합니다.

### 스마트 컨트랙트 컴파일 하기

#### 시작하기 전에

로컬 폴더(예를 들어 `./example/hello/`)에 컨트랙트의 소스 파일이 있어야 합니다. 컨트랙트를 생성하기 위한 자세한 내용은 [Hello World Contract 가이드](https://developers.eos.io/welcome/latest/smart-contract-guides/hello-world)를 확인합니다.

#### 순서

스마트 컨트랙트를을 컴파일 하려면 다음 순서대로 진행합니다.

1. 위에서 예로 들었던 `./examples/hello` 폴더로 이동하여 `./src/hello.cpp` 파일이 있는 것을 확인합니다.
2.  다음 명령을 수행합니다.

    ```cpp
    mkdir build
    cd build
    cmake ..
    make
    ```
3. 다음 두 개의 파일이 생성되었는지 확인합니다.
   * `hello.wasm` : 컴파일 된 wasm 바이너리
   * `hello.abi`: 생성된 ABI 파일

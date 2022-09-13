# Antelop Leap 란?

## Introduction

Antelope Leap(이하 Leap) 은 EOS, Telos, WAX 와 같은 블록체인 네트워크를 구축할 수 있는 오픈소스 소프트웨어 입니다.&#x20;

서두에서 이야기 한 바와 같이 원래 이 소프트웨어는 EOSIO 라는 이름으로 블록체인 기업인 블록원(Block.one) 에서 개발하고 관리 하였습니다. 그러나 2021년 중반, 블록원이 개발을 중단한 이후 EOS 재단인 ENF(EOS Network Foundation) 가 EOSIO 를 포킹(Forking)하여 Mandel 이라는 이름으로 변경하고 개발 및 유지보수를 하였습니다. 그리고 이 소프트웨어를 EOS 네트워크에 적용하기 전, Leap 이라는 이름으로 다시 한 번 리브랜딩 하였습니다.

ENF 는 공식적으로 다음과 같이 Antelope 와 Leap 의 관계를 설명하고 있습니다.

> Antelope is an open framework for building fast, secure, and user-friendly Web3 products and services.
>
> Antelope 은 빠르고 안전하며 사용자 친화적인 Web3 제품과 서비스를 구축할 수 있는 오픈 소스 프레임워크입니다.
>
> Leap is blockchain node software and supporting tools that implements the Antelope protocol.
>
> Leap 은 Antelope 프로토콜을 구현하는 블록체인 노드 소프트웨어와 지원 도구들 입니다.

Leap 은 EOS 뿐만 아니라 WAX, Telos, UX Netork 와 같은 블록체인 네트워크에서도 채택하고 있으며, Leap 의 전신인 EOSIO 는 Fio Protocol, Proton 등과 같은 몇몇 퍼블릭 블록체인 네트워크들이 자신들의 특색에 맞게 커스터마이징 하여 사용하고 있습니다. &#x20;

Leap 이 EOSIO 로 부터 포킹되어 나온지 얼마 되지 않았기 때문에 아직은 두 소프트웨어간의 핵심 기능 차이는 크지 않은 편 입니다. 다만 EOSIO 는 사실상 개발 중단된 상황이고 Leap 은 ENF 의 관리 하에 활발하게 개발 및 유지보수가 이루어지고 있기 때문에 본 문서는 Leap 을 기준으로 작성할 것입니다.

Leap 을 이해하면 EOSIO 을 기반으로 하는 다른 블록체인 네트워크도 쉽게 이해할 수 있으므로, 다른 EOSIO 기반 블록체인에서도 서비스나 Dapp 을 개발하려는 계획을 가지고 있다면, 먼저 Leap 의 기본 및 핵심부터 이해하고 다루는 방법 부터 시작해야 할 것입니다.

## Leap 의 구성 요소

Leap 소프트웨어는 블록체인 네트워크를 운영하고 스마트 컨트랙트를 개발하는데 필요한 여러가지 주요 컴포넌트와 라이브러리들을 가지고 있습니다.

그 중 가장 중요한 핵심 컴포넌트는 다음과 같습니다.

* nodeos
* cleos
* keosd

### nodeos (node + eos)

nodeos 는 Leap 코어 데몬이며 각종 설정을 플러그인 방식으로 구성하여 원하는 목적에 맞는 블록체인 노드를 구성하고 운영할 수 있습니다. nodeos 데몬이 정상적으로 동작하고 있다면, 설정된 내용에 따라 블록을 생성하거나, 트랜잭션을 처리하거나, API 엔드포인트를 제공하거나, 다른 노드와 블록을 동기화 하는 등의 작업을 수행합니다.

### keosd (key + eos + daemon)

로컬 환경에서 개인 키를 저장하고 관리하기 위한 키 관리용 데몬입니다. keosd 는 안전하게 키를 암호화하여 지갑 파일에 저장하는 역할을 합니다.

### cleos (cli + eos)

cleos 는 nodeos 가 제공하는 REST API 와 통신하기 위한 커맨드라인 인터페이스 도구입니다. cleos 를 사용하면 nodeos 에 명령을 전달하거나 스마트 컨트랙트를 배포하는 등의 작업을 할 수 있습니다. 또한 cleos 는 nodeos 뿐만 아니라 keosd 와도 통신 하는데 사용할 수 있습니다.

위 3개 컴포넌트는 Antelope 기반 블록체인을 운영하려면 반드시 알아야 할 요소입니다.

## 스마트 컨트랙트 개발에 필요한 주요 컴포넌트

### CDT

CDT는 WASM(Web Assembly)용 개발 툴킷이자, Antelope 체인 상에서 동작하는 스마트 컨트랙트를 쉽게 작성할 수 있도록 도와주는 도구들을 모은 것입니다. CDT 는 범용 Web Assembly 개발 도구이기도 하지만, Antelope 스마트 컨트랙트를 개발 할 때 Antelope 에 최적화 된 기능도 사용할 수 있습니다. CDT 는 Clang9 을 기반으로 만들어졌으며, 다수의 LLVM 최적화 도구 및 분석 도구를 포함하고 있습니다.

### EOSJS

Leap 에서 제공하는 RPC API를 사용하여 블록체인 네트워크와 통신할 수 있는 프론트엔드 Javascript API SDK입니다.

### D.U.N.E

D.U.N.E (Docker Utilities for Node Execution) 은 Leap 의 프로그램과 CDT 를 비롯한 여러 서비스와 도구들을 추상화해 주는 유틸리티 입니다. D.U.N.E 를 사용하면 노드 관리, 스마트 컨트랙트 컴파일, 테스트, 그밖에 Antelope 스마트 컨트랙트에서 필요한 여러가지 일반적인 작업들을 쉽게 실행할 수 있습니다.

## Leap 개발 생명주기

지금까지 알아본 컴포넌트들의 관계를 그림으로 나타내면 다음과 같습니다.

![](<../.gitbook/assets/image (1).png>)

Antelope Leap 에 대한 핵심 내용은 앞으로 계속하여 컨텐츠로 추가 할 계획이지만, 그 전에 보다 자세한 내용이 필요하시면 Antelope 공식 웹 페이지나 [깃허브 저장소](https://github.com/AntelopeIO) 혹은 최신 자료는 아니지만 [EOSIO 개발자 포털](https://developers.eos.io/)에서도 확인해 보실 수 있습니다.

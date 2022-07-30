# 프로토콜 가이드

Mandel 프로토콜 섹션은 Mandel 플랫폼이 어떻게 동작하고  두 가지 레이어로 나뉠 수 있는지 설명합니다.

Core 는 오픈 소스이며, C++로 작성되어 있습니다. 그리고 System 은 Core 위에서 실행되는 스마트 컨트랙트로 작성되어 있습니다.

## Core

Mandel Core 는 System 레이어에 기본 빌딩 블록을 제공하며 스마트 컨트랙트로 작성되어 있지 않습니다. Core 는 오픈소스로 공개되어 있으며, 소스코드를 원하는 비즈니스 요구사항에 맞추어 커스터마이징 하여 사용할 수 있습니다.

Core  프로토콜 구성 요소는 다음과 같습니다.

1. 합의 프로토콜(Consensus Protocol)
2. 트랜잭션 프로토콜(Transaction Protocol)
3. 네트워크 또는 p2p  프로토콜(Network or Peer to Peer Protocol)
4. 계정과 권한 (Account and Permissions)

## System

Mandel 블록체인 플랫폼은 그 위에 구축된 블록체인이 유연하다는 특성이 있기 때문에 개별 비즈니스 요구사항에 맞게 변경하거나 완전히 수정할 수도 있습니다. 합의, 수수료 일정, 계정 생성 및 수정, 토큰 이코노믹스, BP 등록, 투표, 다중서명 등 핵심 블록체인 기능은 Mandel 플랫폼을 사용하여 구축된 블록체인 위에 구현됩니다. 이러한 기능들을 구현하는 스마트 컨트랙트를 시스템 컨트랙트(System Contract)라고 하고 시스템 컨트랙트로 구현된 계층을 Mandel 시스템 계층, 또는 단순하게 시스템 계층이라고 합니다.

시스템 컨트랙트는 다음과 같습니다.

1. eosio.bios
2. eosio.system
3. rosio.msig
4. eosio.token
5. eosio.wrap

또한 시스템 레이어는 다음 개념들을 포함합니다.

1. System accounts
2. RAM
3. CPU
4. NET
5. Stake
6. Vote

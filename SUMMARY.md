# Table of contents

* [📝 For Leap Developers.](README.md)

## Antelope Leap 개요 <a href="#basic-antelope-leap" id="basic-antelope-leap"></a>

* [Antelop Leap 란?](basic-antelope-leap/what-is-antelope-leap.md)
* [Leap 소프트웨어 설치](basic-antelope-leap/install-leap-software.md)
* [Nodeos 기초](basic-antelope-leap/basic-nodeos.md)
* [Cleos 기초](basic-antelope-leap/basic-cleos.md)
* [Keosd 기초](basic-antelope-leap/basic-keosd.md)
* [계정과 권한 기초](basic-antelope-leap/basic-accounts-and-permissions.md)
* [키 쌍 만들기](basic-antelope-leap/how-to-create-key-pair.md)
* [지갑 만들기](basic-antelope-leap/how-to-create-a-wallet.md)
* [키를 지갑으로 가져오기](basic-antelope-leap/how-to-import-a-key-to-a-wallet.md)
* [계정 만들기](basic-antelope-leap/how-to-create-account.md)
* [계정 정보 확인하기](basic-antelope-leap/how-to-get-account-information.md)

## 스마트 컨트랙트 개발을 위한 환경 구성 <a href="#smart-contracts-dev-environment" id="smart-contracts-dev-environment"></a>

* [개발 환경 개요](smart-contracts/setup-dev-env.md)
* [스마트 컨트랙트 개발의 기본 흐름](smart-contracts-dev-environment/contract-dev-workflow/README.md)
  * [Hello World Contract!](smart-contracts-dev-environment/contract-dev-workflow/hello-world-contract.md)
  * [eosio.token: 토큰 배포, 발행 및 전송](smart-contracts-dev-environment/contract-dev-workflow/token-deploy-issue-transfer.md)
* [CDT(Contract Development Toolkit) 개요와 설치 방법](smart-contracts-dev-environment/cdt-contract-development-toolkit/README.md)
  * [스마트 컨트랙트 컴파일](smart-contracts-dev-environment/cdt-contract-development-toolkit/how-to-compile-smart-contract.md)

## Smart Contract Advanced

* [스마트 컨트랙트 개발자 핸드북](smart-contract-advanced/smart-contract-dev-handbook.md)
* [ABI 파일의 이해](smart-contract-advanced/abi-explained.md)
* [데이터 지속성(Data Persistence)](smart-contract-advanced/antelope-leap-dune/data-persistence.md)
* [보조 인덱스(Secondary Indices)](smart-contract-advanced/secondary-indices.md)
* [스마트 컨트랙트에서 사용자 인증 방법](smart-contract-advanced/smart-contract-authorization.md)
* [인라인 액션을 추가하는 법](smart-contract-advanced/how-to-add-inline-actions.md)
* [다른 스마트 컨트랙트의 인라인 액션을 호출하는 방법](smart-contract-advanced/how-to-call-inline-action.md)
* [컨테이너 기반 Antelope 노드 실행 도구 D.U.N.E](smart-contract-advanced/antelope-leap-dune/README.md)
* [사용자 지정 권한(Custom permission) 연결: 스마트 컨트랙트에서 사용자 지정 권한 설정](smart-contract-advanced/connect-to-customer-permission.md)
* [모범 사례](smart-contract-advanced/best-practices.md)
* [기능(Features)](smart-contract-advanced/features.md)
* [액션 래퍼 사용법](smart-contract-advanced/how-to-use-action-wrapper.md)
* [액션으로부터 값을 반환하는 방법](smart-contract-advanced/how-to-return-from-action.md)
* [지불 가능한 액션(Payable Action)](smart-contract-advanced/payable-action.md)

## Antelope Leap Advanced

* [Leap Advanced](antelope-leap-advanced/leap-advanced.md)
* [스냅샷이나 블록로그를 사용하여 블록체인을 재생하는 방법](antelope-leap-advanced/how-to-replay-blockchain-using-shapshot-or-blocklog.md)
* [스마트 컨트랙트의 보이지 않는 곳에서 일어나는 일](smart-contracts/what-happens-behind-smart-contract.md)
* [Leap 소스코드를 컴파일 후 인스톨](basic-antelope-leap/install-leap-software/leap-1.md)
* [Nodeos Storage 와 Read mode 에 대하여](basic-antelope-leap/nodeos-storage-read-mode.md)
* [테스트넷에 연결된 노드 만들기(작업중)](antelope-leap-advanced/local-node-connected-to-testnet.md)
* [Nodeos Plugin 옵션 상세](basic-antelope-leap/install-leap-software/nodeos-plugin-details/README.md)
  * [Chain Plugin](basic-antelope-leap/install-leap-software/nodeos-plugin-details/chain-plugin.md)
  * [Net Plugin](basic-antelope-leap/install-leap-software/nodeos-plugin-details/net-plugin.md)
  * [Producer Plugin](basic-antelope-leap/install-leap-software/nodeos-plugin-details/producer-plugin.md)
  * [HTTP Plugin](basic-antelope-leap/install-leap-software/nodeos-plugin-details/http-plugin.md)
  * [HTTP Client Plugin](basic-antelope-leap/install-leap-software/nodeos-plugin-details/http-client-plugin.md)
  * [State History Plugin](basic-antelope-leap/install-leap-software/nodeos-plugin-details/state-history-plugin.md)
  * [Resource Monitor Plugin](basic-antelope-leap/install-leap-software/nodeos-plugin-details/resource-monitor-plugin.md)
  * [Trace API Plugin](basic-antelope-leap/install-leap-software/nodeos-plugin-details/trace-api-plugin.md)
  * [Txn Test Gen Plugin](basic-antelope-leap/install-leap-software/nodeos-plugin-details/txn-test-gen-plugin.md)
  * [Login Plugin](basic-antelope-leap/install-leap-software/nodeos-plugin-details/login-plugin.md)
* [Producer 노드 설정](basic-antelope-leap/setup-producer-node.md)
* [체인베이스 설정](basic-antelope-leap/chainbase-setting.md)
* [프로토콜 가이드](basic-antelope-leap/protocol-guide/README.md)
  * [트랜잭션 프로토콜(작업중)](basic-antelope-leap/protocol-guide/transaction-protocol.md)
  * [합의 프로토콜(Consensus Protocol)](basic-antelope-leap/protocol-guide/consensus-protocol.md)
  * [계정과 권한(작업중)](basic-antelope-leap/protocol-guide/account-and-permissions.md)
  * [Network Peer 프로토콜(작업중)](basic-antelope-leap/protocol-guide/network-peer-protocol.md)
* [블록 생성에 대한 상세](basic-antelope-leap/block-producing-explained.md)
* [Nodeos 의 동작 방식](basic-antelope-leap/nodeos-how-to-work.md)
* [트랜잭션 생애주기](basic-antelope-leap/transaction-lifecycle.md)
* [데이터 설계](basic-antelope-leap/data-design.md)
* [스마트 컨트랙트의 퍼포먼스와 스케일링](antelope-leap-advanced/performance-and-scaling.md)
* [Context-Free Data(CFD)란?](basic-antelope-leap/context-free-data-cfd.md)
  * [CFD 정리하는 방법](antelope-leap-advanced/context-free-data-cfd/how-to-prune-cfd.md)
* [바이오스 부트 시퀀스](basic-antelope-leap/bios-boot-sequence.md)
* [스마트 컨트랙트 보안](smart-contracts/smart-contract-security.md)
* [스마트 컨트랙트 메모리](antelope-leap-advanced/smart-contract-memory.md)
* [EOS 지갑의 가져오기 형식](antelope-leap-advanced/eos-wallet-import-format.md)

## Advanced Smart Contract Tutorial

* [Reference](advanced-smart-contract-tutorial/reference.md)
* [License](advanced-smart-contract-tutorial/license.md)
* [Change Log](advanced-smart-contract-tutorial/change-log.md)

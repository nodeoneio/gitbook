---
cover: >-
  https://images.unsplash.com/photo-1508921108053-9f757ead871c?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHwxfHxjJTJCJTJCfGVufDB8fHx8MTY2MzY4MjA2NQ&ixlib=rb-1.2.1&q=80
coverY: -1387.1617113850618
---

# 📝 Introduction

## 시작하기에 앞서...

최근 몇 년간 블록체인 기술은 엄청난 속도로 발전해 왔습니다. 그에 맞춰 여러 블록체인들이 경쟁적으로 나아가고 있지만, EOS 의 기반을 이루는 블록체인 소프트웨어인 EOSIO 는 개발사인 블록원(Block.one)이 개발에서 손을 뗀 후 오랜 기간 기술적으로 정체되어 있었습니다.

한 때 빛나던 최고의 기술력이라는 수식어가 점차 퇴색되어가던 중, 2021년 말 부터 EOS 는 이브 라 로즈(Yves La Rose)가 이끄는 ENF(EOS Network Foundation) 안에 새로운 둥지를 틀게 되었고 다시 비상 할 준비를 하고 있습니다.

ENF 는 2021년 8월 이후로 개발이 중지되었던 EOSIO 의 코드를 포킹(forking) 하여 Mandel 이라는 코드네임으로 개명한 뒤 커뮤니티, BPs, 개발자들과 합심하여 그동안 멈춰 있던 소프트웨어 개발과 유지보수 작업에 박차를 가하기 시작하였습니다.

그리고 컨센서스 업그레이드를 약 한 달여 남긴 8월 17일, ENF 는 Mandel 로 불리던 블록체인 프로토콜의 이름을 정식으로 Antelope 이라는 이름으로, 그리고 특별히 이름이 없었던 프로토콜의 C++ 구현체 소프트웨어 이름을 Leap 으로 리브랜딩하여 공개하였습니다.

그리고 9월 21일 마침내 성공적으로 EOS 메인넷을 Antelope 기반으로 전환하였습니다.

## 그러나...

소프트웨어의 개발은 이렇게 착실히 진행되어가고 있지만 불행히도 개발자들을 위한 문서는 아직 많이 부족합니다. [Antelope Leap 의 공식 깃허브 저장소](https://github.com/AntelopeIO/leap)에서 제공하는 문서들은 아직 많이 빈약하고, 이미 유지보수를 멈춘 [블록원의 개발자 포털](https://developers.eos.io/)에는 아직 많은 정보들이 남아 있지만 오래되어 변경되거나 잘못된 정보들 또한 적지 않습니다.&#x20;

ENF 에서도 자체적으로 [개발자 가이드 문서](http://docs.eosnetwork.com/)를 만들었지만 아직은 내용이 빈약하고 한국의 개발자를 위하여 한글화 된 최신 정보는 거의 없다시피 한 게 현실입니다.

이 모든 점들이 EOS 에서 일어나는 변화들을 보며 관심을 갖고 진입하려는 한국의 신규 개발자들에게 높은 허들이 되고 있습니다.

## 한글화 된 문서

그리하여 노드원에서는 EOS 를 비롯한 Antelope 블록체인 프로토콜을 베이스로 하는 블록체인 네트워크에서의 서비스나 디앱을 구상하는 개발자들을 위하여 한글화된 기술 문서를 구축하려 합니다.&#x20;

초기에는 기초적인 컨텐츠와 함께 블록원의 개발자 포털 자료나 해외 개발자들의 자료를 가져와 적절히 번역/편집하는 정도의 문서를 제공하려는 계획입니다. 그러나, 점차 노드원 독자의 컨텐츠를 추가하고 블록체인 기반 서비스를 개발하는데 필요한 정보를 보다 체계적으로 구성된 문서를 구축함으로서, 한국의 개발자들이 보다 쉽게 Antelope 기반 블록체인에 접근할 수 있게 하고 결과적으로 EOS 생태계가 확장되고 번성하는데 기여하고자 합니다.

본 문서는 완성된 버전이 아니기 때문에 아직 문서 구성이 체계적이지 못하고 아직 완성되지 못했거나 미처 업데이트가 이루어지지 않은 컨텐츠가 존재합니다. 때문에 공개 후에도 계속하여 컨텐츠를 추가하고 업데이트 할 것입니다. 보다 쉽고 정확하고 효율적으로 학습할 수 있는 문서를 작성하려면 개발자 여러분의 적극적인 피드백 또한 필요합니다.

만약 본 문서에서 잘못된 부분이나 오타/오역 등을 발견하시거나 문의사항, 건의사항 등이 있다면 담당자 [이메일](mailto:junhank@nodeone.io) 또는 [노드원 텔레그램](https://t.me/nodeone\_group) 채널로 연락 주시기 바랍니다.

## 문서의 구성

본 문서는 다음과 같이 구성되어 있습니다.

*   Antelope 개요

    기본적인 Leap 소프트웨어(nodeos, cleos, keosd) 개요 및 설치법, 키, 지갑, 트랜잭션, 계정, 권한 등등에 대한 내용을 다룹니다.
* 스마트 컨트랙트 개발을 위한 환경 구성\
  로컬노드를 구성하여 스마트 컨트랙트를 개발할 수 있는 환경을 만들어 봅니다. CDT 설치하고 아주 간단한 스마트 컨트랙트를 작성하고 컴파일 및 배포, 실행, 테스트 해 보는 기본적인 절차를 진행해 볼 것입니다.
* Smart Contract Advanced\
  인라인 액션, 인덱스(Primary/Secondary), ABI, 리카르디안 컨트랙트, 액션 래퍼, 반환값 처리 개요를 다룹니다.
* Antelope Leap Advanced\
  Antelope 에 대하여 보다 심화된 내용을 다룹니다.\
  Antelope 프로토콜 상세, Nodeos 구성 파일 옵션 상세, 체인베이스 DB 상세, 트랜잭션 생애주기, 계정과 권한, CFD 바이오스 부트 시퀀스 등
* (TBD) Advanced Tutorial\
  Hello World 보다 조금 더 많은 기능을 가진 컨트랙트를 작성해 보겠습니다.
* References
* License
* Change Log

문서의 구성은 추후 변경될 수 있습니다.

## 참조 문서

본 문서는 다음 문서들을 기반으로 작성하였으며 각 문서에 명시된 라이선스를 따르고 있습니다.

* [https://github.com/EOSIO/eos/blob/master/LICENSE](https://github.com/EOSIO/eos/blob/master/LICENSE)
* [https://github.com/AntelopeIO/leap/blob/main/LICENSE](https://github.com/AntelopeIO/leap/blob/main/LICENSE)

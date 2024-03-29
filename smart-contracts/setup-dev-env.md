---
description: Development Environment Overview
---

# 개발 환경 개요

Antelope 기반 블록 체인의 로컬 개발 환경이란, Antelope Leap 소프트웨어와 함께 의존성을 가진 소프트웨어들을 설치하여, 시스템 요구 사항을 충족시킨 로컬 시스템을 의미합니다. 그 다음 `nodeos` 를 실행하여 로컬 블록체인 인스턴스를 실행하면 스마트 컨트랙트를 배포할 준비가 된 것입니다.

Antelope 은 Leap 4.0 버전 기준으로 `Ubuntu 18.04 LTS, 20.04 LTS, 22.04 LTS` 만 정식으로 지원합니다. 기타 유닉스 기반 OS 에서도 동작 할 수도 있지만 정상적인 동작을 보장하지는 않습니다.

따라서 본 문서의 스마트 컨트랙트 개발 튜토리얼을 진행하려면 VM 이나 컨테이너 혹은 퍼블릭 클라우드 인스턴스에 Antelope 를 지원하는 Ubuntu 리눅스를 설치하고 노드를 설정해야 합니다.

Leap 소프트웨어를 인스톨하려면 다음 문서를 참조합니다.

{% embed url="https://app.gitbook.com/s/YZT0OiBQKuAU7OjJoCgQ/~/changes/04VA08GIu5KbFDAYakz8/basic-antelope-leap/install-leap-software" %}
Leap 소프트웨어 설치
{% endembed %}

참고로 ENF 는 개발자들의 편의를 위하여 로컬 컴퓨터에서 컨테이너 기반으로 간단히 스마트 컨트랙트 개발 환경을 구축할 있는 유틸리티 소프트웨어인 D.U.N.E 을 만들어 공개하였습니다.

개발자본 문서의 튜토리얼을 학습한 후 어느 정도 Antelope 노드 사용법과 스마트 컨트랙트 개발에 대한 기본에 익숙해졌다면 이 도구를 이용하여 개발 환경을 구축하는 것을 추천합니다. 설정 방법과 간단한 사용법을 [컨테이너 기반 Antelope 노드 실행 도구 D.U.N.E ](../smart-contract-advanced/antelope-leap-dune/)에서 다루고 있으니 이를 참조하여 개발 환경을 구축할 수 있습니다.

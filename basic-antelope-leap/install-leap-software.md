---
description: Install Leap Software
---

# Leap 소프트웨어 설치

블록체인을 사용하는 무언가를 만들 때는 메인넷이든, 테스트넷이든 로컬 환경이든 반드시 동작하고 있는 블록체인 노드가 있어야 합니다.

예를 들어 DApp 을 개발할 때, EOS 정글넷 같은 공개된 테스트넷의 노드가 제공하는 RPC API 를 사용할 수도 있지만, 그보다는 먼저 로컬 환경에서 동작하는 테스트용 노드를 구성하는것이 여러모로 편리합니다. 따라서 가장 먼저 해 볼 것은 로컬 환경에서 Antelope 노드를 구동하기 위한 Leap 소프트웨어를 설치해 보는 것입니다.

Leap 4.0 버전 기준으로 공식적으로 지원하는 OS 는 다음과 같습니다.

* Ubuntu 18.04 (Leap 4.0 까지만 지원하므로 가급적 20.04 이후 버전을 권장합니다.)
* Ubuntu 20.04
* Ubuntu 22.04

기타 POSIX 기반 OS 에도 설치될 수 있을지도 모르지만 공식적으로 지원하지는 않기 때문에 올바른 동작을 보장하지는 않습니다. 따라서 다른 OS 에서 Antelope 노드를 설정하려면 위와 같이 지원되는 버전의 Ubuntu 환경을 컨테이너나 VM 을 사용하여 구성해야 합니다.

개발을 위한 시스템 최소 사양이나 권장 사양은 특별히 공개된 내용이 없지만, 컨테이너나 VM을 구성할 수 있는 일반적인 개발용 기기라면 문제 없이 로컬 노드를 가동할 수 있습니다. 또는 개발용으로만 사용한다는 전제로 AWS, Azure, GCP 같은 퍼블릭 클라우드 서비스에 인스턴스를 만들어 노드를 구성할 수도 있습니다.

Leap 소프트웨어를 설치하려면 다음과 같은 방법을 이용할 수 있습니다.

* [Leap 바이너리 인스톨](install-leap-software.md#leap)
* [Leap 소스코드를 컴파일 후 인스톨](install-leap-software.md#leap-1)
* [로컬 환경에 D.U.N.E 설정](install-leap-software.md#d.u.n.e)

## Leap 바이너리 인스톨

미리 만들어진 바이너리 파일을 인스톨하여 빠르게 노드를 만들 수 있습니다. Leap 의 바이너리는 Antelope 의 공식 Github 에서 데비안 소프트웨어 패키지(.deb) 형태로 제공됩니다.

Antelope 학습 초기부터 어느 정도 익숙해질 때 까지는 네이티브 또는 VM 환경에 바이너리를 설치하여 사용하는것을 추천합니다. Leap 의 바이너리는 다음의 공식 깃허브 릴리스 페이지에서 찾아볼 수 있습니다. 설치하려는 환경에 맞는 데비안 패키지를 다운로드 받아 설치합니다.

{% embed url="https://github.com/AntelopeIO/leap/releases" %}

다음은 Ubuntu 22.04 에 Leap v4.0.0 을 설치하는 예제입니다.

{% code overflow="wrap" %}
```
$ wget https://github.com/AntelopeIO/leap/releases/download/v4.0.0/leap_4.0.0-ubuntu22.04_amd64.deb
$ sudo apt install ./leap_4.0.0-ubuntu22.04_amd64.deb
```
{% endcode %}

설치가 완료되면 다음과 같이 확인할 수 있습니다.

```
$ nodeos --version
v4.0.0
```

## Leap 소스코드를 컴파일 후 인스톨(준비중)

보다 고급 개발자들을 위한 인스톨 방법입니다. 다음 링크에서 확인할 수 있습니다.

{% embed url="https://app.gitbook.com/s/YZT0OiBQKuAU7OjJoCgQ/~/changes/GyFj0wmAOGq0NW7Uht4a/antelope-leap-advanced/leap" %}

## 로컬 환경에 DUNES 설정

DUNES 은 로컬 머신 환경에서 간편하게 Docker 컨테이너 기반 개발용 블록체인 노드 환경을 설정 할 수 있게 도와주는 도구입니다. Antelope Leap 의 사용법에 어느 정도 익숙해졌다면 편리하게 사용할 수 있습니다.

D.U.N.E 의 설정 방법은 링크에서 확인할 수 있습니다.

{% embed url="https://app.gitbook.com/s/YZT0OiBQKuAU7OjJoCgQ/~/changes/GyFj0wmAOGq0NW7Uht4a/smart-contracts-dev-environment/antelope-leap-dune" %}

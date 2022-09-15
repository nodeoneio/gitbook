# Leap 소프트웨어 설치

Leap 노드를 Antelope 소프트웨어를 설치하고 노드를 설정하는 방법은 다음과 같습니다.

* 바이너리 인스톨
* 소스코드 컴파일 후 인스
* 로컬 환경에서 D.U.N.E 설치

## 바이너리 인스톨

빠르게 Antelope 노드를 설정하려면 미리 만들어진 바이너리 파일을 인스톨 할 수 있습합니다. Antelope 의 바이너리는 공식 Github 에서 데비안 소프트웨어 패키지(.deb) 형태로 제공됩니다. 최신 버전의 Antelope 을 다음 링크에서 확인할 수 있습니다.

{% embed url="https://github.com/eosnetworkfoundation/Antelope/releases" %}

## 소스 코드 컴파일 후로 인스톨

보다 고급 개발자들을 위한 인스톨 방법입니다. 다음 링크에서 확인할 수 있습니다.

{% embed url="https://developers.eos.io/manuals/eos/latest/install/build-from-source/index" %}

## 로컬 환경에서 D.U.N.E 설치

D.U.N.E 는 로컬 환경에서 간편하게 개발용 블록체인 노드를 설정 할 수 있게 도와주는 도구입니다. 다음 링크에서 D.U.N.E 의 설정 방법을 확인할 수 있습니다.

{% embed url="https://app.gitbook.com/s/YZT0OiBQKuAU7OjJoCgQ/~/changes/nLRGL2We77qm5hvw33Tk/Antelope-d.u.n.e" %}

## 설치 확인

인스톨이 완료되면 다음과 같이 정상 설치 여부를 확인합니다.

```
nodeos --version
v3.1.0-rc2
```

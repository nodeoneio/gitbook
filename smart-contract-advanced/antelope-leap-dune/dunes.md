---
description: DUNES Plugins
---

# DUNES 플러그인

DUNES 사용자는 플러그인을 사용하여 기능을 확장할 수 있습니다. DUNES 플러그인은 간단한 파이썬 스크립트로, 다음의 조건을 충족해야 합니다.

## 플러그인 요구사항

1. 플러그인 코드는 `../src/plugiun/` 아래에 위치해야 합니다.
2. 위 디렉토리에 `main.py` 라는 스크립트를 작성합니다.
3. `main.py` 에 다음과 같은 3개의 함수를 작성합니다.
   * `add(parsing(parser)` - `argparse.ArgumentParser` 의 인스턴스를 받는 함수입니다. 인수를 파싱하여 새로운 DUNES 명령어를 추가하는 데 사용됩니다.&#x20;
   * (선택사항) `set_dunes(dunes)` - DUNES 의 인스턴스를 받는 함수로, 사용자가 플러그인 상에서 DUNES 를 사용할 수 있습니다. 나중에 DUNES 를 사용할 일이 있다면 이 함수를 사용할 수 있습니다.
   * `handle_args(args)` - `ArgumentParser.pare_args` 가 반환한 네임스페이스를 받는 함수.  이 함수는 새로운 DUNES 명령어 인수를 다룰 때 사용합니다.

## 플러그인 예제

[플러그인 예제 페이지](https://github.com/AntelopeIO/DUNES/blob/main/plugin\_example)를 보시면 여러가지 예제를 찾아볼 수 있습니다. 예제 플러그인을 테스트 하려면 [../plugin\_example/](https://github.com/AntelopeIO/DUNES/blob/main/plugin\_example) 를 복사하거나 심볼릭 링크를 만들어 `../src/plugin/` 에 갖다 붙입니다. 이렇게 하면 DUNES 가 자동으로 새 플러그인을 찾아냅니다.

### dunes\_hello

아주 간단한 플러그인으로, `--hello` 를 DUNES 명령어에 추가합니다. 그래서 `dunes --hello` 를 실행하면 예제 출력이 화면에 나타납니다.

### account\_setup

이 플러그인은 `--bootstrap-account` 를 DUNES 명령에 추가합니다. 이를 실행하면 3개의 예제 계정(alice, bob, cindy)이 추가됩니다. 또한 `eosio.token` 컨트랙트가 위 계정들에게 배포됩니다.

이 예제에서 어떻게 `set_dunes` 함수를 사용하여 `dunes` 인스턴스를 저장한 다음 계정들을 생성하고 설정하는지 확인할 수 있습니다. &#x20;

## 자세한 구현 방법

DUNES 는 시작할 때 `src/plugin` 의 하위 디렉토리에 있는 플러그인을 자동으로 찾아 낸 다음, 각각의 `main.py` 파일을 동적으로 로딩합니다. 각 플러그인의 함수는 다음과 같은 순서로 호출됩니다.

1. `add_parsing(parser)` - 파싱된 인수를 추가하기 위해 가장 먼저 호출되는 함수. 사용자는 이 단계에서 플러그인을 초기화 할 수 있지만, 이 시점에서 해당 플러그인을 사용하게 될 지는 알 수 없다는 것에 주의해야 합니다.&#x20;
2. (선택사항) `set_dunes(dunes)` - DUNES 의 인스턴스를 사용하고 싶다면 이 함수에 DUNES  객체를 저장할 수 있습니다.
3. `handle_args(args)` - 사용자는 파싱된 인수가 사용되고 있는지, 함수 내에서 핸들링 하고 있는지 체크해야 합니다. 이것이 플러그인이 실제로 하는 일을 기술하는 메인 함수입니다.  보통 이 함수 내에서 DUNES 객체가 필요합니다.

# 키를 지갑으로 가져오기

이 단원에서는 keosd 디폴트 지갑으로 개인키를 가져오는 방법을 설명합니다. 여기에 저장된 개인키는 Antelope 블록체인의 트랜잭션을 인증하는데 사용합니다.

## 시작하기 전에

* 본 단원을 학습하려면 `cleos` 가 설치되어 있어야 합니다. `cleos` 는 `leap` 소프트웨어를 설치할 때 `nodeos, keosd` 와 함께 설치됩니다.
* 잠금 해제된 지갑이 있어야 합니다. 지갑을 만드는 방법은 [지갑 만들기](https://app.gitbook.com/s/YZT0OiBQKuAU7OjJoCgQ/\~/changes/9SOrppvOOBlShstEZa2F/basic-antelope-leap/how-to-create-a-wallet) 단원을 확인합니다.
* `cleos wallet import`  명령을 사용할 것입니다.
* 공개키와 개인키로 이루어진 비대칭 암호화 키 쌍에 대한 기초적인 이해가 필요합니다.

## 순서

* 가져오려는 개인 키를 복사합니다.
* 다음 명령을 입력하면 디폴트 지갑에 개인 키를 가져다 넣을 수 있습니다.

```
cleos wallet import
private key:
```

위와 같이 `private key:` 프롬프트가 나타나면 복사한 개인 키를 붙여넣고 엔터 키를 누릅니다.

개인키 가져오기에 성공하면 다음과 같이 개인 키의 쌍이 되는 공개키가 화면에 표시됩니다.

```
imported private key for: EOS8FBXJUfbANf3xeDWPoJxnip3Ych9HjzLBr1VaXRQFdkVAxwLE7
```

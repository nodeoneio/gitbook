# 키 쌍 만들기

## 개요

본 단원에서는 Antelope 블록체인에서 트랜잭션을 서명할 때 사용하는 공개키와 개인키의 비대칭 암호화 키 쌍을 만들어보겠습니다.

## 시작하기 전에

본 단원을 학습하려면 `cleos` 가 설치되어 있어야 합니다. `cleos` 는 `leap` 소프트웨어를 설치할 때 `nodeos, keosd` 와 함께 설치됩니다.

## 순서

다음 방법으로 키 쌍을 만든 다음 콘솔에 출력하거나 파일로 저장할 수 있습니다.

* 키 쌍을 만들고 콘솔에 출력합니다.

```
cleos create key --to-console

# 출력 결과 예시
Private key: 5KPzrqNMJdr6AX6abKg*******************************cH
Public key: EOS4wSiQ2jbYGrqiiKCm8oWR88NYoqnmK4nNL1RCtSQeSFkGtqsNc
```

* 키 쌍을 만들고 결과를 파일로 저장합니다.

```
cleos create key --file keypair.txt
saving keys to keypair.txt
```

* 결과를 다음과 같이 확인할 수 있습니다.

```bash
cat pw.txt
Private key: 5K7************************************************
Public key: EOS71k3WdpLDeqeyqVRAAxwpz6TqXwDo9Brik5dQhdvvpeTKdNT59

```

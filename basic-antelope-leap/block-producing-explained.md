---
description: Block Producer Explained
---

# 블록 생성에 대한 상세

단순하게 설명하기 위하여 다음과 같은 표기법을 사용하겠습니다.

```
m = max_block_cpu_usage
t = block-time
e = last-block-cpu-effort-percent
w = block_time_interval = 500ms
a = produce-block-early-amount = (w - w*e/100) ms
p = produce-block-time; p = t - a
c = billed_cpu_in_block = minimum(m, w - a)
n = network tcp/ip latency
```

비슷한 하드웨어/eosio 버전/설정 에 대한 피어 검증은 ≤m

아래 다이어그램에 표시된 것과 같이 네 개의 BP를 가진 네트워크 토폴로지를 예로 들어 설명하겠습니다.

![](<../.gitbook/assets/image (4).png>)

* BP-A 는 p시간에 블록을 전송한다.
* BP-B는 t시간에 블록이 필요하며 그렇지 않으면 드랍시킨다.
* 만약 BP-A 가 다음과 같이 12개의 블록(block, b)을 특정 시간(time, t)에 생성하고 있다면,\
  1, bt 1.5, bt 2, bt 2.5, bt 3, bt 3.5, bt 4, bt 4.5, bt 5, bt 5.5, bt 6, bt 6.5\
  BP-B 는 bt 6.5 에 대하여 6.5 time 만큼 필요하게 되고 0.5 를 위해 최종적으로 bt 7이 필요하게 된다.
* bt 7에서 0.5를 뺀 시간은 bt 6.5의 시간과 같으므로 시간 t는 BP-A의 마지막 블록 시간이며 BP-B가 첫 번째 블록을 시작해야 하는 시간이다.

#### 예제 1

BP-A가 50% e 를 가짐, m = 200ms, c = 200ms, n = 0ms, a = 250ms:

BP-A-Peer (t-250ms)에 전송 <-> BP-A-Peer 는 200ms 동안 처리 후 (t - 50ms)에 전송 <-> BP-B-Peer 200ms 동안 처리 후 (t + 150ms)에 전송 <-> BP-B에는 150ms 늦게 도착. 너무 늦음.

#### 예제 2

BP-A가 40% e를 가짐, m = 200ms, c = 200ms, n = 0ms, a = 300ms:

(t-300ms) <->(+200ms) <->(+200ms) <->BP-B에 100ms 늦게 도착. 너무 늦음.

#### 예제 3

BP-A는 30% e를 가짐, m = 200ms, c = 150ms, n = 0ms, a = 350ms:

(t-350ms) <->(+150ms) <->(+150ms) <->BP-B에 50ms 여유롭게 도착.

#### 예제 4

BP-A가 25% e를 가짐, m = 200ms, c = 125ms, n = 0ms, a = 375ms:

(t-375ms) <->(+125ms) <->(+125ms) <->(+125ms) <-> BP-B에 125ms 여유롭게 도착.

#### 예제 5

BP-A가 10% e를 가짐, m = 200ms, c = 50ms, n = 0ms, a = 450ms:

(t-450ms) <->(+50ms) <->(+50ms) <->BP-B에 350ms 여유롭게 도착.

#### 예제 6

BP-A가 10% e를 가짐, m = 200ms, c = 50ms, n = 15ms, a = 450ms:

(t-450ms) <- +15ms ->(+50ms) <- +15ms ->(+50ms) <- +15ms -> BP-B 에 305ms 여유롭게 도착.

#### 예제 7

월드와이드 네트워크의 예는 다음과 같습니다.

BP-A가 10% e 를 가짐, m = 200ms, c = 50ms, n = 15ms/250ms, a = 450ms:

(t-450ms) <- +15ms -> (+50ms) <- +250ms -> (+50ms) <- +15ms -> BP-B 에 70ms 여유롭게 도착.

p2p 노드에서 `wasm-runtime=eos-vm-jit, eos-vm-oc-enabled` 를 설정하면 검증 시간이 단축된다.

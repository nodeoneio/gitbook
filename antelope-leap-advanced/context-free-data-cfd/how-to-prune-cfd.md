# CFD 정리하는 방법

## 개요

이 단원에서는 트랜잭션에서 컨텍스트 프리 데이터(CFD)를 정리하는 방법을 알아볼 것입니다. 이 프로세스에는 `--prune-transactions` 옵션과 함께 `eosio-blocklog` 를 사용하는 방법, 컨텍스트 프리 데이터를 포함하는 트랜잭션 ID, 그리고 아래와 같이 추가 옵션을 사용하는 방법도 알아보겠습니다.

{% hint style="info" %}
트랜잭션 데이터를 정리하는 것은 Antelope 네트워크상에서 다수 BP가 사전에 합의하지 않는 한 퍼블릭 Antelope 블록체인에는 적합하지 않습니다. 퍼블릭 Antelope 네트워크의 생산자 노드가 트랜잭션에서 컨텍스트 프리 데이터를 제거하더라도 해당 노드에만 적용이 되며 블록체인의 무결성은 훼손되지 않을 것입니다.
{% endhint %}

## 사전 요구사항

절차를 시작하기 전에 다음 항목을 미리 알고 있거나 완료해야 합니다.

* 컨텍스트 프리 데이터를 가지고 있는 완료된(Finalized) 블록에 포함된, 만료된 트랜잭션 ID
* [컨텍스트 프리 데이터](../../basic-antelope-leap/context-free-data-cfd.md) 의 개념을 알고 있어야 합니다.
* ``[`eosio-blocklog`](https://developers.eos.io/manuals/eos/latest/utilities/eosio-blocklog) 명령줄 도구 사용법을 확인합니다.

## 순서

다음은 트랜잭션에서 컨텍스트 프리 데이터를 삭제하는 순서입니다.

1. 컨텍스트 프리 데이터를 잘라낼 트랜잭션 ID를 찾습니다(예: `<trx_id>`). 트랜잭션 ID는 트랜잭션의 `id` 필드에서도 찾을 수 있습니다.
2. 트랜잭션이 포함된 블록 번호(예: `<block_num>`)를 찾습니다. 블록 번호가 트랜잭션의 `block_num` 필드와 일치하는지 확인합니다.
3. 블록 디렉토리 및 상태 기록 디렉토리(해당하는 경우)를 찾습니다(예: `<blocks_dir>` 및 `<state_hist_dir>`).&#x20;
4. 다음과 같이 `eosio-blocklog` 도구를 실행합니다.\
   `eosio-blocklog [--blocks-dir <blocks_dir>] [--state-history-dir <state_hist_dir>] --prune-transactions --block-num <block_num> --transaction <trx_id> [--transaction <trx_id2> ...]`

다음은 명령이 성공했을 때 적용되는 내용입니다.

* `eosio-blocklog` 유틸리티는 오류 코드 0(오류 없음)로 자동으로 종료됩니다.&#x20;
* 블록 로그의 정리된 트랜잭션 내에서 다음 필드가 업데이트됩니다.
  * 0 이었던 `prunable_data["prunable_data"][0]` 필드가 1로 설정됩니다.&#x20;
  * `signatures` 필드가 빈 배열로 설정됩니다.&#x20;
  * `context_free_data` 필드가 빈 배열로 설정됩니다.&#x20;
  * `packed_context_free_data` 필드가 있다면 제거됩니다.&#x20;

다음은 실패했을 때 적용되는 내용입니다.

* `eosio-blocklog` 유틸리티는 `stderr`에 오류를 출력하고 0이 아닌 오류 코드로 종료됩니다.

## 주의사항

몇 가지 추가 고려 사항이 있습니다.

* 같은 블록 내의 다수의 트랜잭션을 `eosio-blocklog` 에 전달할 수 있습니다.&#x20;
* `eosio-blocklog` 를 사용하여 삭제된 트랜잭션을 가지고 있는는 블록을 표시할 수 있습니다.

## 예제&#x20;

위에서 다루었던 내용을 아래의 예제에서 확인해 보겠습니다.

### 컨텍스트 프리 데이터를 가진 트랜잭션 예제(처리전)

```
{
  "id": "1b9a9c53f9b692d3382bcc19c0c21eb22207e2f51a30fe88dabbb45376b6ff23",
  "trx": {
    "receipt": {
      "status": "executed",
      "cpu_usage_us": 155,
      "net_usage_words": 14,
      "trx": [
        1,
        {
          "compression": "none",
          "prunable_data": {
            "prunable_data": [
              0,
              {
                "signatures": [
                  "SIG_K1_K3AJXEMFH99KScLFC1cnLA3WDnVK7WRsS8BtafHfP4VWmfQXXwX21KATVVtrCqopkcve6V8noc5bS4BJkwgSsonpfpWEJi"
                ],
                "packed_context_free_data": "0203a1b2c3031a2b3c"
              }
            ]
          },
          "packed_trx": "ec42545f7500ffe8aa290000000100305631191abda90000000000901d4d00000100305631191abda90000000000901d4d0100305631191abda900000000a8ed32320000"
        }
      ]
    },
    "trx": {
      "expiration": "2020-09-06T02:01:16",
      "ref_block_num": 117,
      "ref_block_prefix": 699066623,
      "max_net_usage_words": 0,
      "max_cpu_usage_ms": 0,
      "delay_sec": 0,
      "context_free_actions": [
        {
          "account": "payloadless",
          "name": "doit",
          "authorization": [],
          "data": ""
        }
      ],
      "actions": [
        {
          "account": "payloadless",
          "name": "doit",
          "authorization": [
            {
              "actor": "payloadless",
              "permission": "active"
            }
          ],
          "data": ""
        }
      ],
      "signatures": [
        "SIG_K1_K3AJXEMFH99KScLFC1cnLA3WDnVK7WRsS8BtafHfP4VWmfQXXwX21KATVVtrCqopkcve6V8noc5bS4BJkwgSsonpfpWEJi"
      ],
      "context_free_data": [
        "a1b2c3",
        "1a2b3c"
      ]
    }
  },
  "block_time": "2020-09-06T02:00:47.000",
  "block_num": 119,
  "last_irreversible_block": 128,
  "traces": [
    {
      "action_ordinal": 1,
      "creator_action_ordinal": 0,
      "closest_unnotified_ancestor_action_ordinal": 0,
      "receipt": {
        "receiver": "payloadless",
        "act_digest": "4f09a630d4456585ee4ec5ef96c14151587367ad381f9da445b6b6239aae82cf",
        "global_sequence": 156,
        "recv_sequence": 2,
        "auth_sequence": [],
        "code_sequence": 1,
        "abi_sequence": 1
      },
      "receiver": "payloadless",
      "act": {
        "account": "payloadless",
        "name": "doit",
        "authorization": [],
        "data": ""
      },
      "context_free": true,
      "elapsed": 206,
      "console": "Im a payloadless action",
      "trx_id": "1b9a9c53f9b692d3382bcc19c0c21eb22207e2f51a30fe88dabbb45376b6ff23",
      "block_num": 119,
      "block_time": "2020-09-06T02:00:47.000",
      "producer_block_id": null,
      "account_ram_deltas": [],
      "account_disk_deltas": [],
      "except": null,
      "error_code": null,
      "return_value_hex_data": ""
    },
    {
      "action_ordinal": 2,
      "creator_action_ordinal": 0,
      "closest_unnotified_ancestor_action_ordinal": 0,
      "receipt": {
        "receiver": "payloadless",
        "act_digest": "b8871e8f3c79b02804a2ad28acb015f503e7f6e56f35565e5fa37b6767da1aa5",
        "global_sequence": 157,
        "recv_sequence": 3,
        "auth_sequence": [
          [
            "payloadless",
            3
          ]
        ],
        "code_sequence": 1,
        "abi_sequence": 1
      },
      "receiver": "payloadless",
      "act": {
        "account": "payloadless",
        "name": "doit",
        "authorization": [
          {
            "actor": "payloadless",
            "permission": "active"
          }
        ],
        "data": ""
      },
      "context_free": false,
      "elapsed": 11,
      "console": "Im a payloadless action",
      "trx_id": "1b9a9c53f9b692d3382bcc19c0c21eb22207e2f51a30fe88dabbb45376b6ff23",
      "block_num": 119,
      "block_time": "2020-09-06T02:00:47.000",
      "producer_block_id": null,
      "account_ram_deltas": [],
      "account_disk_deltas": [],
      "except": null,
      "error_code": null,
      "return_value_hex_data": ""
    }
  ]
}
```

### 순서

위의 트랜잭션 예제를 사용하여 순서대로 진행해 보겠습니다.

1. 트랜잭션 ID: `1b9a9c53f9b692d3382bcc19c0c21eb22207e2f51a30fe88dabb45376b6ff23`을 찾습니다.&#x20;
2. 블록 번호 `119`를 찾습니다.&#x20;
3. 블록 디렉토리 및 상태 기록 디렉토리(해당하는 경우)를 찾습니다(예: `<blocks_dir>` 및 `<state_hist_dir>`).&#x20;
4. 다음과 같이 `eosio-blocklog` 도구를 실행합니다.\
   `eosio-blocklog --blocks-dir <blocks_dir> --state-history-dir <state_hist_dir> --prune-transactions --block-num 119 --transaction 1b9a9c53f9b692d3382bcc19c0c21eb22207e2f51a30fe88dabbb45376b6ff23`

성공하면 별다른 출력 없이 `eosio-blocklog`가 종료되고 실패하면 stderr에 오류가 출력됩니다.

### 컨텍스트 프리 데이터를 가진 트랜잭션 예제(처리 후)

다시 한번 트랜잭션을 조회하면 다음과 같이 표시됩니다.

```
{
  "id": "1b9a9c53f9b692d3382bcc19c0c21eb22207e2f51a30fe88dabbb45376b6ff23",
  "trx": {
    "receipt": {
      "status": "executed",
      "cpu_usage_us": 155,
      "net_usage_words": 14,
      "trx": [
        1,
        {
          "compression": "none",
          "prunable_data": {
            "prunable_data": [
              1,
              {
                "digest": "6f29ea8ab323ffee90585238ff32300c4ee6aa563235ff05f3c1feb855f09189"
              }
            ]
          },
          "packed_trx": "ec42545f7500ffe8aa290000000100305631191abda90000000000901d4d00000100305631191abda90000000000901d4d0100305631191abda900000000a8ed32320000"
        }
      ]
    },
    "trx": {
      "expiration": "2020-09-06T02:01:16",
      "ref_block_num": 117,
      "ref_block_prefix": 699066623,
      "max_net_usage_words": 0,
      "max_cpu_usage_ms": 0,
      "delay_sec": 0,
      "context_free_actions": [
        {
          "account": "payloadless",
          "name": "doit",
          "authorization": [],
          "data": ""
        }
      ],
      "actions": [
        {
          "account": "payloadless",
          "name": "doit",
          "authorization": [
            {
              "actor": "payloadless",
              "permission": "active"
            }
          ],
          "data": ""
        }
      ],
      "signatures": [],
      "context_free_data": []
    }
  },
  "block_time": "2020-09-06T02:00:47.000",
  "block_num": 119,
  "last_irreversible_block": 131,
  "traces": [
    {
      "action_ordinal": 1,
      "creator_action_ordinal": 0,
      "closest_unnotified_ancestor_action_ordinal": 0,
      "receipt": {
        "receiver": "payloadless",
        "act_digest": "4f09a630d4456585ee4ec5ef96c14151587367ad381f9da445b6b6239aae82cf",
        "global_sequence": 156,
        "recv_sequence": 2,
        "auth_sequence": [],
        "code_sequence": 1,
        "abi_sequence": 1
      },
      "receiver": "payloadless",
      "act": {
        "account": "payloadless",
        "name": "doit",
        "authorization": [],
        "data": ""
      },
      "context_free": true,
      "elapsed": 206,
      "console": "Im a payloadless action",
      "trx_id": "1b9a9c53f9b692d3382bcc19c0c21eb22207e2f51a30fe88dabbb45376b6ff23",
      "block_num": 119,
      "block_time": "2020-09-06T02:00:47.000",
      "producer_block_id": null,
      "account_ram_deltas": [],
      "account_disk_deltas": [],
      "except": null,
      "error_code": null,
      "return_value_hex_data": ""
    },
    {
      "action_ordinal": 2,
      "creator_action_ordinal": 0,
      "closest_unnotified_ancestor_action_ordinal": 0,
      "receipt": {
        "receiver": "payloadless",
        "act_digest": "b8871e8f3c79b02804a2ad28acb015f503e7f6e56f35565e5fa37b6767da1aa5",
        "global_sequence": 157,
        "recv_sequence": 3,
        "auth_sequence": [
          [
            "payloadless",
            3
          ]
        ],
        "code_sequence": 1,
        "abi_sequence": 1
      },
      "receiver": "payloadless",
      "act": {
        "account": "payloadless",
        "name": "doit",
        "authorization": [
          {
            "actor": "payloadless",
            "permission": "active"
          }
        ],
        "data": ""
      },
      "context_free": false,
      "elapsed": 11,
      "console": "Im a payloadless action",
      "trx_id": "1b9a9c53f9b692d3382bcc19c0c21eb22207e2f51a30fe88dabbb45376b6ff23",
      "block_num": 119,
      "block_time": "2020-09-06T02:00:47.000",
      "producer_block_id": null,
      "account_ram_deltas": [],
      "account_disk_deltas": [],
      "except": null,
      "error_code": null,
      "return_value_hex_data": ""
    }
  ]
}
```

위 결과에서 다음과 같은 내용이 변경된 것을 확인할 수 있습니다.

* `prunable_data["prunable_data"][0]` 필드가 1이 되었습니다.
* `signature` 필드가 빈 배열이 되었습니다.
* `context_free_data` 필드가 빈 배열이 되었습니다.
* `packed_context_free_data` 필드가 제거되었습니다.

## 다음으로 해야 할 일

컨택스트 프리 데이터를 정리한 후에 다음과 같을 작업을 할 수 있습니다.

* 트랜잭션을 정리한 이후 컨텍스트 프리 데이터가 포함되어 있는지 확인합니다.&#x20;
* 블록 로그에서 정리된 트랜잭션을 포함하는 블록을 표시합니다.

# Near light client on MAPO blockchain

## Contract Address

[Here to get Near mainnet and testnet light client contract address.](/develop/light-client/README.md)


## Contract interface

```go

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

interface ILightNode {
    function verifyProofData(bytes memory _receiptProof) external view returns (bool success, bytes memory logs);

    function updateBlockHeader(bytes memory _blockHeader) external;

    function headerHeight() external view returns(uint256);
}
```

## Interact with contract interface

### update_block_header

Get the last block header of each block, verify the signatures and update validators for the next epoch.

#### input parameters

| parameter    | type  | comment                                            |
| ------------ | ----- | -------------------------------------------------- |
| _blockHeader | bytes | borsh encode of the last block header of one epoch |

### verify_proof_data

Verify the specified receipt is in the specified block.

#### input parameters

| parameter     | type  | comment                                                                                                     |
| ------------- | ----- | ----------------------------------------------------------------------------------------------------------- |
| receipt_proof | bytes | abi encode of borth encode of the proof of one receipt and borsh encode of the specified blockblock header |

### headerHeight

Get the latest block header number the light client has updated.

#### output parameters

| type    | comment                                                     |
| ------- | ----------------------------------------------------------- |
| uint256 | the latest block header number the light client has updated |

## Data structure

### Here are some main data structure for near light client contract.

#### blockHeader

```go
    struct LightClientBlock {
        bytes32 prev_block_hash;
        bytes32 next_block_inner_hash;
        BlockHeaderInnerLite inner_lite;
        bytes32 inner_rest_hash;
        OptionalBlockProducers next_bps;
        OptionalSignature[] approvals_after_next;
        bytes32 hash;
        bytes32 next_hash;
    }   
```

```go
    struct BlockHeaderInnerLite {
        uint64 height; // Height of this block since the genesis block (height 0).
        bytes32 epoch_id; // Epoch start hash of this block's epoch. Used for retrieving validator information
        bytes32 next_epoch_id;
        bytes32 prev_state_root; // Root hash of the state at the previous block.
        bytes32 outcome_root; // Root of the outcomes of transactions and receipts.
        uint64 timestamp; // Timestamp at which the block was built.
        bytes32 next_bp_hash; // Hash of the next epoch block producers set
        bytes32 block_merkle_root;
        bytes32 hash; // Additional computable element
    }
```

```go
    struct OptionalBlockProducers {
        bool some;
        BlockProducer[] blockProducers;
        bytes32 hash; // Additional computable element
    }
```

```go
 struct OptionalSignature {
        bool some;
        Signature signature;
    }
```

```go
    struct BlockProducer {
        PublicKey publicKey;
        uint128 stake;
        // Flag indicating if this validator proposed to be a chunk-only producer (i.e. cannot become a block producer).
        bool isChunkOnly;
    }
```

```go
    struct PublicKey {
        bytes32 k;
    }
```

```go
    struct Signature {
        bytes32 r;
        bytes32 s;
    }
```

#### Transaction Outcome Proofs

```go
    struct FullOutcomeProof {
        ExecutionOutcomeWithIdAndProof outcome_proof;
        MerklePath outcome_root_proof; // TODO: now empty array
        BlockHeaderLight block_header_lite;
        MerklePath block_proof;
    }
```

```go
    struct ExecutionOutcomeWithIdAndProof {
        MerklePath proof;
        bytes32 block_hash;
        ExecutionOutcomeWithId outcome_with_id;
    }
```

```go
    struct MerklePath {
        MerklePathItem[] items;
    }
```

```go
    struct BlockHeaderLight {
        bytes32 prev_block_hash;
        bytes32 inner_rest_hash;
        NearDecoder.BlockHeaderInnerLite inner_lite;
        bytes32 hash; // Computable
    }
```

```go
    struct ExecutionOutcomeWithId {
        bytes32 id; // The transaction hash or the receipt ID.
        ExecutionOutcome outcome;
        bytes32 hash;
    }
```

```go
    struct ExecutionOutcomeWithId {
        bytes32 id; // The transaction hash or the receipt ID.
        ExecutionOutcome outcome;
        bytes32 hash;
    }
```

```go
    struct ExecutionOutcome {
        bytes[] logs; // Logs from this transaction or receipt.
        bytes32[] receipt_ids; // Receipt IDs generated by this transaction or receipt.
        uint64 gas_burnt; // The amount of the gas burnt by the given transaction or receipt.
        uint128 tokens_burnt; // The total number of the tokens burnt by the given transaction or receipt.
        bytes executor_id; // Hash of the transaction or receipt id that produced this outcome.
        ExecutionStatus status; // Execution status. Contains the result in case of successful execution.
        bytes32[] merkelization_hashes;
    }
```

```go
    struct ExecutionStatus {
        uint8 enumIndex;
        bool unknown;
        bool failed;
        bytes successValue; // The final action succeeded and returned some value or an empty vec.
        bytes32 successReceiptId; // The final action of the receipt returned a promise or the signed transaction was converted to a receipt. Contains the receipt_id of the generated receipt.
    }
```

```go
    struct MerklePathItem {
        bytes32 hash;
        uint8 direction; // 0 = left, 1 = right
    }
```

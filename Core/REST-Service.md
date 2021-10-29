Synchronous Queries
===================

Blockchain state can be quered via GET REST API service.

Response payload is packed using `MessagePack` format and sent as plain bytes
with `Content-Type: application/octet-stream` header.

Currently the service allows to fetch the following information:

- transaction receipt and content.
- block info (header).
- account/wallet info.

## Getting transaction content

Request URL: `/api/v1/transaction/<ticket>`

#### Input:

- ticket: transaction ticket as given by the submit service response.

#### Output:

```json
[
  [
    account,
    nonce,
    network,
    target,
    method,
    [
      signatureType,
      pubKeyValue
    ],
    args
  ],
  signature
]
```

In other words it is the very same structure as described
[here](Transaction-Format.md) but without keys.

NULL if ticket not found.

## Getting transactions receipt

Request URL: `/api/v1/receipt/<ticket>`

#### Input:

- ticket: transaction ticket as given by the submit response.

#### Output:

```json
[
  height
  index
  success
  returns
]
```

- height: Number. Block number where the transaction has been inserted.
- index: Number. Transaction offset within the block.
- success: Boolean. True if the smart contract has been executed successfully.
- returns: Binary. Smart contract specific output data.

NULL if ticket not found.

## Getting Block info

Request URL: `/api/v1/block/<height>`

#### Input:

- height: offset within the blockchain)

#### Output:

```json
[
  height,
  txs_count,
  prev_hash,
  txs_hash,
  rec_hash,
  state_hash,
]
```

- height: Number. Block offset within the blockchain.
  Genesis block has height 0.
- txs_count: Number. Number of transactions within this block.
- prev_hash: Bytes. Hash of the previous block.
- txs_hash: Bytes. Merkle tree root of the transactions contained
  within this block.
- rec_hash: Bytes. Merkle tree root of the receipts contained
  within this block.
- state_hash: Bytes. Merkle tree root of all the accounts (aka world state).

NULL if not found.

## Getting account info

Request URL: `/api/v1/account/<id>`

#### Input:

- id: account identifier as described [here](Account-Identifier.md).

#### Output:

```json
[
  id,
  assets,
  contract_ref,
  data_hash
]
```

- id: String. Account identifier.
- assets: map of asset name and it's smart contract-specific values
  (e.g. <"BTC", bytes>).
- contract_ref: Bytes. Smart contract idenfifier associated to the account.
  It is defined as the hash of the WASM sources.
- data_hash: Bytes. Merkle tree root of the data associated with the account.ss

## Session Example

### Submit Transaction

- type: POST
- path: /api/vi/submit

Request body:

```json
{
  "data": {
    "account": "QmYHnEQLdf5h7KYbjFPuHSRk2SPgdXrJWFh5W696HPfq7i",
    "nonce": "a03a5500433cf88c",
    "network": "skynet",
    "target": "87b6239079719fc7e4349ec54baac9e04c20c48cf0c6a9d2b29b0ccf7c31c727",
    "method": "transfer",
    "caller": {
      "type": "ecdsa_p384r1",
      "value": "7cUWbhbYtrVqMpEyLXYh3RDAYgpNGzPFoA55ymm7NndERKxebZKMgXo35k3T45uZrm2S4uw5k6aaYG96J4e2tWumVY4p1yv6hgSspSHVzvMKadYCHZByscm1oknaZjC86FF6"
    },
    "args": {
      "to": "QmTuhaS8rBRjBSxPYHGVGtZmmkN3fVtHJTuTbwwUSdnB8a",
      "amount": 19,
      "asset": "WPC"
    }
  },
  "signature": "MvbDchmXV1aHytKWr91aPCwsWAatmFG6fCusLmyJY6FcVf5ZZB4TjUhboXteYDqDB4brAegZDH9rfBoHVTqihGscLnKbcLmGuoKWVu92uuX24MdK4ijKdiEx58E9QZpQyqS"
}
```

Response body:

```json
"12205b47bca14ab8cd339481a0717ac0b3f4b6601e8eb73804571b9335276306c791"
```

### Get Transaction

- type: GET
- path: /api/v1/transaction/12205b47bca14ab8cd339481a0717ac0b3f4b6601e8eb73804571b9335276306c791

Response body:

```json

[
  [
    "QmYHn...",
    "a03a5500433cf88c",
    "skynet",
    "87b62390...",
    "transfer",
    [
      "ecdsa_p384r1",
      "7cUWbhbYtrV..."
    ],
    "82fcff2ba7..."
  ],
  "MvbDchmXV1..."
]
```

### Get Receipt

- type: GET
- path: /api/v1/receipt/9QEDuUq12byR74Cf8W7DuUScz6S2fNpc69Kzrmf1ooQV

Response body:

```json
[
  344,
  0,
  true,
  null
]
```

### Get Block

- type: GET
- path: /api/v1/block/344

Response body:

```json
[
  344,
  100,
  "e08ff0f6b4e0738c7b6fd1d033ba021cf1f36f882ff7167aae1e6a3bba10a8b5",
  "1c53e075e5962038dfeb5d2bae0581d4c1445345584168835f2935fdd4c7d880",
  "b2fc1d6be8bf1e55306ec56ab48bece541feec31187fe4438275c1cadaa4be1e",
  "34e3ecda323d9d8fd1b59c6e2026d694d5dd9eb821a57b9416dd3c1159778f16"
]
```

### Get Account

- type: GET
- path: /api/v1/account/QmYHnEQLdf5h7KYbjFPuHSRk2SPgdXrJWFh5W696HPfq7i {

Response body:

```json
[
  "QmYHnEQLdf5h7KYbjFPuHSRk2SPgdXrJWFh5W696HPfq7i",
  {
    "WPC": "94342fff0c"
  },
  "87b6239079719fc7e4349ec54baac9e04c20c48cf0c6a9d2b29b0ccf7c31c727",
  null,
]
```

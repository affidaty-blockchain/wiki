Transactions Format
===================

## Schema

```json
  {
    "data": {
        "account": string,
        "nonce": bytes,
        "network": string,
        "contract": bytes,
        "method": string,
        "caller": {
          "type": string,
          "curve": string,
          "value": bytes,
        },
        "args": bytes,
    },
    "signature": bytes
  }
```

- `account`: target account identifier (as defined [here](Trinci2/Core/Account-Identifier)).
  This is the account over which the invoked contract method will be executed.

- `nonce`: pseudo-random value for anti-replay protection. Allows to get a
  different hash for transactions with the very same payload.

- `network`: blockchain network name. Blockchains with different names are
  incompatible. Each node is configured to operate over a specific network.

- `contract`: smart contract identifier defined as the contract `wasm` multihash.

- `method`: name of smart contract's function to execute.

- `caller`: transaction submitter's identity used for digital signature
  verification and (optionally) used by the smart contracts to take
  application-specific decisions:
  - `type`: digital signature algorithm identifier (e.g. "ecdsa").
  - `curve`: ecdsa curve identifier (e.g. "secp284r1").
  - `value`: public key value.

- `args`: smart contract specific arguments. This information is opaque to the
  core and is passed "as-is" to the smart contract.
  Can be anything and is a smart contract duty to decode the content in the way
  that best fits for the application (message-pack by default).

- `signature`: digital signature of the message-packed `data` field.
  Verified using the public key contained in the `data` field itself.

### Example:

```json
  {
    "data": {
      "account": "QmYHnEQLdf5h7KYbjFPuHSRk2SPgdXrJWFh5W696HPfq7i",
      "nonce": 0102030405060708,
      "network": "skynet",
      "contract": 12202c26b46b68ffc68ff99b453c1d30413413422d706483bfa0f9...,
      "method": "terminate",
      "caller": {
        "type": "ecdsa",
        "curve": "secp384r1",
        "value": 045936d631b849bb5760bcf62e0d1261b6b6e227dc0a389...
      },
      args: 82a46e616d65a4436f6c65a5656d61696cb263727573746e3440746f706c...
    },
    "signature": 69bc880acdab862ebbf776b5411b04b5e872369d3b40ca448832...
  }
```

## Data Serialization

Transaction structures are encoded using [MessagePack](https://msgpack.org)
binary format.

Both *anonymous* and *named* serialization formats are supported by the core
during transaction validation.

When using *anonymous* format the fields order matters.

When using named format, fields order is not important. Anyway, to obtain best
performances during deserialization phase, it is good practice to organize the
fields in the same order as described by this document.

### Data Digital Signature

Steps:

1. The `data` field is serialized as a byte array.

   `data_bytes = msgpack.serialize(data);`

2. The bytes are digitally signed using the submitter's private key.

   `signature = ecdsa-sign(submitter-private-key, data_bytes);`

3. The signature field is appended to the previous structure:

```json
  transaction {
    "data": {
      ...
    },
    "signature": ...
  }
```

4. The overall transaction structure is serialized using MessagePack format.

   `tx_bytes = msgpack.serialize(transaction);`

5. The transaction bytes (`tx_bytes`) are finally sent to the blockchain node.

Important: If digital signature has been produced using a serialization format
(*anonymous* or *named*) then the same format shall be used when serializing the
overall transaction for core submission.

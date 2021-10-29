Blockchain Message Format
=========================

Message definition used for communication with the blockchain service using the
channel exposed by the blockchain service.

This format is used by all the components that require some form of
communication with the blockchain.

Both strictly internal components such as the node *bootstrap* component and the
*tracer* and all the components providing an interface to the external world
such as *p2p*, *bridge*, *rest* services are using this message format.

Messages structure is given in a pseudo json/asn1/rustacean format.

When serialization and deserialization is required to be performed by the
blockchain service, the submitter shall pack the structure using "MessagePack"
**anonymous** format.

### Support structures definitions

```
CHOICE : only one element can be selected from the given set.

FLAG : multiple elements can be selected from the given set.

SequenceOf : an array of homogeneous elements.

ByteBuffer := SequenceOf u8

Ticket := MultiHash(MessagePack(Transaction::Data))
```

### Blockchain Message

A message that can be consumed and produced by the blockchain service.

```
Message {
    type: u8,
    body: MessageBody,
}
```

Message body types along with the type tag values used for serialization:

```
MessageBody : CHOICE {
    Exception = 0,
    Subscribe = 1,
    Unsubscribe = 2,
    PutTransactionRequest = 3,
    PutTransactionResponse = 4,
    GetTransactionRequest = 5,
    GetTransactionResponse = 6,
    GetReceiptRequest = 7,
    GetReceiptResponse = 8,
    GetBlockRequest = 9,
    GetBlockResponse = 10,
    GetAccountRequest = 11,
    GetAccountResponse = 12,
    Stop = 254,
    Packed = 255,
}
```

The message body is "flattened" into the `Message` structure, meaning that no
additional nesting level should be applied.

For example, a Subscribe message is serialized as it is defined as:

```
Message {
    type: 1,
    id: String,
    event: Event,
    packed: bool,
}
```

### Stop

Special message to stop the blockchain service.

As a design decision, to prevent external components from stopping the
blockchain service, this message is only accepted when it is coming from an
internal component.

Only `Packed` messages are assumed to come from an external component.

```
Stop
```

### Subscribe

Subscribe to blockchain events.

The subscription allows to receive messages already packed. This option can be
useful for relay services such as p2p and bridge.

```
Subscribe {
    id: String,         // Subscriber id
    event: Event,       // Bitflags with events we want start receiving
    packed: bool,       // Receive already packed data (MessagePack)
}

Event : FLAG {
    Transaction, // receive a `GetTransactionResponse` when a new (good) tx is received
    Block,       // receive a `GetBlockResponse` when a block is generated
    Request,     // receive any request that the blockchain is trying to send out
}
```

### Unsubscribe

Unsubscribe to blockchain events.

```
Unsubscribe {
    id: String,         // Subscriber id
    event: Event,       // Bitflags with events we want stop receiving
}
```

### Put Transaction

Submit a new transactions.

The request can be confirmed or unconfirmed. When confirmation is requested,
the blockchain service will send back a response containing the submit result
corresponding to the transaction.

On success the submit result is the transaction ticket (hash). On failure the
result contains the failure reason as a string.

```
PutTransactionRequest {
    confirmed: bool,        // If we require a confirmation
    tx: Transaction
}

PutTransactionResponse {
    hash: Ticket
}
```

### Get Transaction

Get a transaction given a the transaction ticket.

```
GetTransactionRequest {
    hash: Ticket
}

GetTransactionResponse := {
    tx: Transaction
}
```

### Get Receipt

Get a transaction receipt given a the transaction ticket.

```
GetReceiptRequest := {
    hash: Ticket
}

GetReceiptResponse := {
    rx: Receipt
}
```

### Get Block

Get a block content.

Optionally allows to get the hashes of the transactions executed by the
specified block.

```
GetBlockRequest := {
    height: u64             // Block height (offset from the genesis block)
    txs: bool               // Fetch transactions tickets as well
}

GetBlockResponse := {
    block: Block
    txs: TransactionHashes
}

TransactionsHashes : CHOICE {
    null,                   // Null if `BlockRequest::txs` is false
    SequenceOf Ticket       // Sequence of block transaction tickets
}
```

### Get Account

Get an account content.

```
GetAccountRequest := {
    id: String,               // Account identifier
    data: SequenceOf string   // Account data keys (may be empty)
}

GetAccountResponse := {
    res: Account,
    data: SequenceOf DataResult
}

DataResult : CHOICE {
    ByteBuffer,
    Null
}
```

### Exception

Something went wrong processing a confirmed message. This message contains an
error message along with the source.

```
Exception := {
    kind: string,
    source: string,
}
```

### Packed

Message generally coming from an external source and forwarded "as-is" to the
blockchain service.

The packed message shall recursively contain a serialized `Message` in
MessagePack format. The message is thus deserialized by the blockchain service
before further processing.

A response to a message received as `Packed` is sent as `Packed` as well.

For security reasons this type of message cannot contain a `Stop` message.

```
Packed := ByteBuffer
```

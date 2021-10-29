Account Identifier
==================

Every user account has an identifier generated as a function of the user
PublicKey using the following formula:

```
AccountId = Base58 ( MultihashHeader | SHA256 ( ProtobufHeader | ASN1Header | DecompressedRawPublicKey  )  )
```

Properties:
- bound to the account owner public key.
- generated via a cryptographically secure hash algorithm with collision
  resistance property.
- short and is easily exchanged between blockchain users.
- future-proof since leaves open the opportunity for algorithm upgrade
  (protocol agility).
- aligned to the
  [libp2p](https://github.com/libp2p/specs/pull/100/files#diff-4304afc8ded80c4141b2f42497ea593b513e0323f4734b79ecc264411ece3733)
  PeerId generation method.

### Procedure for ECDSA Key

Raw Decompressed Public Key:

```bash
    04 (decompressed form)
    X: 5936d631b849bb5760bcf62e0d1261b6b6e227dc0a3892cbeec91be069aaa25996f276b271c2c53cba4be96d67edcadd
    Y: 66b793456290609102d5401f413cd1b5f4130b9cfaa68d30d0d25c3704cb72734cd32064365ff7042f5a3eee09b06cc1
```

Prepend ASN.1 header (as defined by x509 standard)

```bash
    30 76   // strucure (bytes count = 118)
      30 10   // structure (bytes len = 16)
        06072a8648ce3d0201      // EC public key OID
        06052b81040022         // secp384r1 curve OID
      03 62 00  // bitstring (bytes count = 98)
        04
        5936d631b849bb5760bcf62e0d1261b6b6e227dc0a3892cbeec91be069aaa25996f276b271c2c53cba4be96d67edcadd
        66b793456290609102d5401f413cd1b5f4130b9cfaa68d30d0d25c3704cb72734cd32064365ff7042f5a3eee09b06cc1
```

Prepend Protobuf header (as defined by libp2p)

```bash
    08 03  // Algorithm type identifier (ECDSA)
    12 78  // Content Length
      3076301006072a8648ce3d020106052b81040022036200
      04
      5936d631b849bb5760bcf62e0d1261b6b6e227dc0a3892cbeec91be069aaa25996f276b271c2c53cba4be96d67edcadd
      66b793456290609102d5401f413cd1b5f4130b9cfaa68d30d0d25c3704cb72734cd32064365ff7042f5a3eee09b06cc1
```

Compute the SHA256 of the overall buffer

```bash
    sha = SHA256(buffer)
    sha = 93d8a1e44a9d179238b598458f5293666890e8c0fcec0cce682522398d726dd5
```

Prepend the "multihash" header to the resulting sha.

```bash
    12      // hash algorithm identifier (0x12 = SHA256)
    20      // hash length (0x20 = 32)
    93d8a1e44a9d179238b598458f5293666890e8c0fcec0cce682522398d726dd5
```

Final result in base58

```bash
    AccountId = "QmYHnEQLdf5h7KYbjFPuHSRk2SPgdXrJWFh5W696HPfq7i"
```

### Multihash Details

Adds to the pure hash a short header with the algorithm identifier and length.

Reference [here](https://github.com/multiformats/multihash)

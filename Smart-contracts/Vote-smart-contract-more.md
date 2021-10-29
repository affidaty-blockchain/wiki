More information about Vote
===
### Creation of the vote token

- _Named User_ **U**: actor requiring, after authentication, a token to vote.
- _Authority_ **A**: actor releasing voting tokens.
- _Anonymous User_ **X**: id of the account from where the user whants to spend its voting token.
- _Polling Station_ **P**: the place where an anonymous user spends his token to vote.

- Token is signed using the authority RSA private key.
- Token is verified by the polling station using the authority RSA public key.

#### Token request procedure using blind signature (off-chain)

The authority has the private key 'd' and a public key '(e, n)' of each polling
station.

Note all the operations are performed modulo 'n' even if not specified.

User request steps:

1. **U** generates a secret account-id **X** from where is intended to vote.
2. **U** requests for ballot config information.
3. **U** receives a list of polling-stations. It polling station **P** is the one at
   position 'X mod K' , with 'K' the number of polling stations in the list.
4. **U** takes the public key '(e, n)' of its polling station.
5. **U** generates a random number 'R' such that 'gcd(R,n) = 1' (coprimes).
6. **U** sends to **A** the blinded request as: 'req = (X \* R^e)'.

Authority response

1. **A** applies the salt 'S' to the user request: 'salted = (req \* S)'.
2. **A** signs the salted request: 'salted^d = (req _ S)^d = (X _ R^e _ S) ^d = (X _ S)^d \* R'
3. **A** sends the signed salted request to U.

Token extraction

1. **U** removes the blinding by multiplying by the inverse of R modulo n (existence
   guaranteed because R and n are coprimes) : 'token = salted^d _ R^-1 = (X _ S)^d _ R _ R^-1 = (X \* S)^d'
2. **U** now has a signed token '(X \* S)^d' to vote from the blockchain account X.

#### Voting procedure (on-chain)

Steps:

1.  Anonymous user **X** sends a transaction (signed by **X**) to vote smart contract along with the signed token.
2.  The core checks that transaction comes from **X** (check performed by the core using ECDSA digital signature).
3.  The contract checks that the token has not been spent before (burned list).
4.  The contract checks that token has a valid signature:
    - decrypts the token using authority public key: `token^d = (X * S)`
    - locally recomputes `X * S` and checks that the result is equal to the decrypted token.

#### Example to check implementation

Notes:

- All the fields shall be loaded within a bigint in big endian.
- All parameters, except the `ACCOUNT-ID` are given as hex strings that shall be
  decoded in a byte array before usage.
- ACCOUNT_ID shall be loaded in a bigint by considering its byte array form:

Example.

```json
    ACCOUNT_ID = "QmYHnEQLdf5h7KYbjFPuHSRk2SPgdXrJWFh5W696HPfq7i"

    as a hex string is: "516d59486e45514c64663568374b59626a4650754853526b325350676458724a5746683557363936485066713769"

    This hex string shall be loaded as all the other parameters.
```

2048-bit values.

```json
    // Public key modulus.
    N = "d0bff10834c15cadceca813f735887e96dc1f85f2d4e3b6b2a21a0388ead2298542a9660ddf383af826f215d55e73fb5f7e4460da4e236b8873675cc88e48e4b48f641b8f650135ab500379b705bc8e2854ccc0b40b941246298668daa5989ad8dc4b0deeebb96a84e8d514442a2a87b0c7d1283317197e5c6e529271336253148e1bfe21348f26defee7a2601bed32033bafde83c09c04db814bdb3d3c0731e479854ebf0134ed0ec2fcbb6d3f60153938f6a455895c1250014e2e964611399352ce231b4ea94e36a82e755f8959d75f9b05ec4f936ee04f72f6c4e63bb322be58bf839cf11f12edaa54df264f4077a4e3b13d4b3b33084e9266a8626452f1b"

    // Public key exponent.
    E = "010001"

    // Private key exponent.
    D = "2182fed5db6a434f9fc800b1c7f9a96ffeefc9c8b5c43e63b8d2c71eed40bc320a29001532ec79f27e150b1d29c243071a71aa115cabf82eed7dffb14334b5e73e225270b81228d941ff73eddf3ecce41894389981dd7ba8b4af549f5f7e9a4ca35ab52e44d3169f9464b96c24ea69f3bf10811d509db4cded7d327b146977d4c46e49bc1969505f3db2a1cad1e2069fe406c11055f49825c97df664a3b11625c72142cd23568e6cb526df600357e8afd31c6987cf1f812b95f478cac75620e0443f2e05b51fca8890958e9514883fc90f601bbd82b8792e92b378d85818387c2fa249b4a92203d0dcaf17ee88301609e5c6ad6e226cc19b673af2667043c3b1"

    // Random integer such that gcd(R, N) = 1
    R = "ce7e97cec1ff7ac0ad973ffb4a4080c18da6dac0d91e35d51e6d89508df49025cd843e769c09711c6912e0d355b5a3a69611e5568950d773cb09be9c94678bfe71ec03fa51c4d0c4b734eb2781a09a050d41ada94b053e38cb7348d29b101e6ec9ca8f85e56174ae60fead6e4a8042a1751f5bc4574190d8a360136aa077f75d8ac4c66ffb54ea80418c4fe5e1aa8fe748f9728f04887d07ca26c38cd7d4e23da129ac983dfea8d6f156a1e51431d2a4b2939acec452f0b630c80d8f777f09ef6ad9867684241fe78f4dcc1f183495f097a5321a94cf9ee6567126c8e1a6674e11cd0c59b71fc661bb3ab1759c75aaaabaf3869180f7972061b486278c41de73"

    // Random integer inverse modulo N.
    I = "838ffb8cae6d3a6576d73652e5cb47609e82abed8cbb1df4bc9fb51c88449f57bff39708e35d8ded47cca1d08039c13ed77fd6846817bcf42195626e71d10ed2ef234a7fdc9e26cf035e1d143cce7cae7a3ce5b722b9cb434850bda638d960a20b76cee9cf750f7deda2d69ab6bff5f0171edf14bf1585e297ab8ff30e9fd3a31dee008f128398210c3b2d17f124dc2799a5ca0a05f03154f9c1a9c407b5bce088e55a68c4cedfcae2bbc739f339c440d1dc8e7c10b1ce9617f714e13ac5f574a4638559c25e4e6d53bc4ccff2720168e5f916b6f3a171b0122c29d90a50c939f45bd60e92bc0a99f1b6f29323a47056140add2dad6a20ab47cd55b33ba5167e"

    // Authority salt.
    S = "000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f"

    // Voting account-id.
    ACCOUNT_ID = "QmYHnEQLdf5h7KYbjFPuHSRk2SPgdXrJWFh5W696HPfq7i"

    // Signed token.
    TOKEN = "1482d74e08e38a5841a116b84eb0c51997b3c26664fdc7188f2bfc68a6fe66abb50f8845a63ba48c0fdea6dae30b00b822117102ff089b193a4c675b47f7fa6bb3dc4f576cee0076e410683212fe2d41d49966da936b17aeda647fdf4f73730da475f27359b91cd990095d002409ee82bd1f194fa6a5d5ebe94b30375cb396ef7fbc027efb3e119a79e002ba672314b722e1773a013e1913c2f17692be9e00cf403cdc08c4395fc5f122656d40aa702a4797e55d09a00bc03e0c821a0ea7d6248dc7db5f9832b4d6e0efa7bc966a407355233f25b4f6caad303a1418d7c5ca3a873558bacf534220af5d3d4dc00e3f6c3c482557415e3416fb7de254c8e811b4"
```

512 bit values.

```json
    // Public key modulus.
    N = "db651a5584bc01af06507acac6fe2730ad135cd567ed92963123f64e951ee953d45b64e4eba2e88b0a3902f343bbf32983d43571aecc89b954996feb5260e11f";

    // Public key exponent.
    E = "010001";

    // Private key exponent.
    D = "3bfffb62b5e940c0a006747e6e4b65665f7ef31d7dcdb01019224fa3100f231a034dea0569f603584557acbb0bfc702b99b33dea2e07ce8e8217e0d89bb0b869";

    // Random integer such that gcd(R, N) = 1
    R = "e141b9fa127e0bdef1c9466674a8dcd14951f858f63d997887d16223d1f1c6e7b6f0a47ca48fc3993603851606aa49703ccdf35485705ea1eaefd8e5f832d83b";

    // Random integer inverse modulo N.
    I = "1da54c4624613375aa8016aceae9adc6158b3868ec8f1fb88fb12a4c60c04c8ead292be7deffaa377c88dc8c923f8d8903ec6f9a97647917757f767241014bfa";

    // Authority salt.
    S = "000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f"

    // Voting account-id.
    ACCOUNT_ID = "QmYHnEQLdf5h7KYbjFPuHSRk2SPgdXrJWFh5W696HPfq7i"

    // Signed token.
    TOKEN: &str = "a8cb0cb91a102c9a9a101b1bd5d77c89b097723c41d0a40d573525cb9eb1bdbaaa5db804baf526d88f2989b2257e09f0dc5bebd72b2d6add6d617b747ae2dc98";
```

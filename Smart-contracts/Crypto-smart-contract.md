Crypto
===

## Features
 - Allows to use some cryptographic methods

## Methods

### `hash`
 - Calculates the hash of the data passed in the args
 - Returns the hash as binary

 ```json
    args: {
        "algorithm": string,
        "data": binary,
    }
  ```

### `verify`
 - Verifies the data signature

 ```json
    args: {
        "pk": PublicKey,
        "data": binary,
        "sign": binary,
    }
  ```

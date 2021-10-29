Storage
===

## Features

- Management of account storage
- Exchange of [TAI](Misc/Trinci-Application-Interface.md) compliant assets.
- Creation of a new account.

## Methods

### `transfer`

- Allows the account owner to transfer an asset to another account.
  If the destination account has not been already taken, then it creates it.

  ```json
  args: {
      "to": account-id,    // Destination account id
      "asset": account-id, // Asset account id
      "units": integer,    // Amount (in asset units)
  }
  ```

### `store_data`

- Allows the owner to store arbitrary binary data with a specified `key` on the account
  Can be executed only by the account owner.

  ```json
  args: {
      "key": string,
      "data": binary,
  }
  ```

### `load_data`

- Allows the owner to load binary data previously stored with a specified `key` from the account
  Can be executed only by the account owner.

  ```json
  args: {
      "key": string,
  }
  ```

### `remove_data`

- Allows the owner to delete binary data previously stored on the account with a specified `key`
  Can be executed only by the account owner.

  ```json
  args: {
      "key": string,
  }
  ```

### `balance`

- Allows the owner to know the amount of the request asset
  Can be executed only by the account owner.

  ```json
  args: {
      "asset": account-id,
  }
  ```

  Returns the balance, e.g.:

  ```json
  {
      42
  }
  ```

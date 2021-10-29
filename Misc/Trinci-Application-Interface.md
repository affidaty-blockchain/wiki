Trinci Application Interface
======================


### Trinci Application Interface (TAI)

To ensure the usability of a certain asset on smart contracts designed by third
parties it is necessary that the asset implements a known interface.

This interface (TAI) request the implementation of the following methods:

- `transfer(from, to, units)`: transfer the given number of asset units from one
  account to another.
  - _Note:_ the `from` account in explicit (some account could be authorized to
    perform operations on other accounts, depends by the contract).
- `stats()`: returns a list of key-value caracteristics such as max-units,
  working-units, owner-id, etc.

_Note:_

- The `mint` operation is implicitly performed with a transfer where the `from`
  field is `null`.
- The `burn` operation is implicitly performed with a transfer where the `to`
  field is `null`.

### Asset method invocation:

```rust
   sdk::call(asset_id, asset_app_hash, method, args)
```

- _asset_id_: asset account-id
- _app_hash_: smartcontract hash (can be empty)
- _method_: method to call (e.g. `transfer`)
- _args_: arguments

### Asset Privileges

When an asset execute a method on its context every method acquire privileges to
modify the content of each account (relatively to the current asset).

**SECURITY-EXTENSION**: If an asset is not registered on the `Service` Account a
user can receive this asset only if he has previously enabled the asset calling
a transfer of 0 units of the asset in question.

---

## Methods (TAI)

### `transfer`

```json
   args: {
       "to": account-id,
       "amount": integer
   }
```

### `mint`

```json
   args: {
       "to": account-id,
       "amount": integer
   }
```

### `burn`

```json
   args: {
       "to": account-id,
       "amount": integer
   }
```

### `stats`

```json
   {
      "minted_units": integer,
      "max_units": integer,
      ...
   }
```

## Asset Locking

Actions on asset could be forbidden by the account Owner, the Contract or the
asset Creator.

The strenght order of lock from lowest to highest is as follows:

```bash
LockPrivilege : { Owner | Contract | Creator }
```
e.g.: an asset locked by the Owner could be overwhelmed by a Contract or the
asset Owner.

Te operativity on the asset can be forbidden in one direction or both:

```bash Locktype : { None | Deposit | Withdraw | Full } ```

### Escrow Contract - An asset lock practical example

- The `init` method for the escrow contract lock the withdraw of the asset
  `(LockPrivilege: Contract, LockType: Withdraw)` both in case that the
  `customer` has already made the deposit or in case there are not enough
  funds.
- The customer, once the escrow is initialized, can deposit the amount of asset
  required but cannot withdraw any amount from the escrow account despite
  having the private key.
- Only the `guarantor` can automatically unlock the asset (and finish the
  escrow) with an `update` transaction.

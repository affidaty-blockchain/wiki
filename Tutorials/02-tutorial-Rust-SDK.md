Trinci _rust_ SDK
===

## How to write a method callable from a transaction

A method to be executed through a transaction:

- must have this signature:
  ```rust
  fn my_method(context: AppContext, args: SerializableStruct) -> WasmResult<Value>;
  ```
  - where `SerializableStruct` is a struct that derives the `Serializable`/`Deserializable` traits from the [serde_derive](https://crates.io/crates/serde_derive) crate.
  ```rust
  #[derive(Serialize, Deserialize, Debug)]
  #[cfg_attr(test, derive(PartialEq, Clone, Default))]
  struct SerializableStruct {
      pub field1: u32,
      pub field2: u8,
      ...
  }
  ```
- must be exported by the `app_export!` sdk macro, like this:
  ```rust
  sdk::app_export!(..., my_method, ...);
  ```

## Host Functions Wrapper

### _log_

```rust
pub fn log(msg: &str);
```
- send the message `msg` to the host to be logged.


### _load_data_

```rust
pub fn load_data(key: &str) -> Vec<u8>;
```
- load the serialized data stored at the given `key` from the current account.

### _store_data_

```rust
pub fn store_data(key: &str, buf: &[u8]);
```

- store the _data_ passed as bytes array in the current account at the given _key_.


### _remove_data_

```rust
pub fn remove_data(key: &str);
```

- remove the data stored at the given `key` from the current account.

### _load_asset_
```rust
pub fn load_asset(id: &str) -> Vec<u8>;
```
- load asset from the account `id` specified.
- the `asset id` is the current `account id`
  
### _store_asset_
```rust
pub fn store_asset(id: &str, value: &[u8]);
```
- store asset to the account `id` specified as byte array.
- the `asset id` is the current `account id`

### _verify_
```rust
pub fn verify(pk: &PublicKey, data: &[u8], sign: &[u8]) -> bool;
```
 - verify the signature of the given data by the given pk and algorithm

### _call_
```rust
pub fn call(account: &str, method: &str, data: &[u8]) -> WasmResult<Vec<u8>>;
```
- calls a method of an arbitrary smart contract passing the data as argument
- if succeed returns the data from the method called

### _asset_balance_
```rust
pub fn asset_balance(asset: &str) -> WasmResult<u64>;
```
- get account balance for a given `asset` id.
- this is an helper function over the lower level `call(asset_id, "balance", args)`.

### _asset_transfer_
```rust
pub fn asset_transfer(from: &str, to: &str, asset: &str, units: u64) -> WasmResult<()>;
```
- transfer an amount of asset `units` to a destination account.
- this is an helper function over the lower level `call(asset_id, "transfer", args)`.

### _asset_lock_
```rust
pub fn asset_lock(asset: &str, to: &str, value: LockType) -> WasmResult<()>;
```
- Lock/Unlock the asset.
- this is an helper function over the lower level `call(asset_id, "lock", true/false)`.


### _load_asset_typed_
```rust
pub fn load_asset_typed<T: DeserializeOwned + Default>(id: &str) -> T;
```
- load asset with from the given account id and tries to convert it into a type.

### _store_asset_typed_
```rust
pub fn store_asset_typed<T: Serialize>(id: &str, value: T);
```
- store the typed asset to the given account id.


## RAW Host Functions
The functions above are wrapper to the raw `host_function` that can be also called directly.
- These functions require data structures to be written to memory and passed their address and size.
- `WasmSlice` that is a i64 where is coded `address` and `size` of a memory location that stores the result.

### _hf_log_
- Allows to log strings
```rust
fn hf_log(address: i32, size: i32);
```
Example of usage:
```rust
...
let msg = String::from("Hello");
let buf = msg.as_bytes();

hf_log(buf.as_ref() as i32, buf.len() as i32);

...
```

### _hf_load_data_
- Load data from the account
```rust
fn hf_load_data(key_addr: i32, key_size: i32) -> WasmSlice;
```


### _hf_store_data_
- Store data to the account
```rust
fn hf_store_data(key_addr: i32, key_size: i32, data_addr: i32, data_size: i32);
```

### _hf_remove_data_
- Remove data from the account
```rust
fn hf_remove_data(key_addr: i32, key_size: i32);
```

### _hf_load_asset_
- load asset from the given account `id`
```rust
fn hf_load_asset(id_addr: i32, id_size: i32) -> WasmSlice;
```

### _hf_store_asset_
- store asset to the given account `id`
```rust
fn hf_store_asset(id_addr: i32, id_size: i32, value_addr: i32, value_size: i32);
```

### _hf_verify_
- verify the data signature
```rust
    fn hf_verify(
        pk_addr: i32,
        pk_size: i32,
        data_addr: i32,
        data_size: i32,
        sign_addr: i32,
        sign_size: i32,
    ) -> i32;
```

### _hf_call_
- call the method of another account
```rust
    fn hf_call(
        account_addr: i32,
        account_size: i32,
        method_addr: i32,
        method_size: i32,
        data_addr: i32,
        data_size: i32,
    ) -> WasmSlice;
```

## Utility Functions

### _rmp_serialize_named_
- Serialize a type implementing `Serialize` trait using MessagePack format with named keys.
```rust
pub fn rmp_serialize_named<T>(val: &T) -> WasmResult<Vec<u8>>
```

### _rmp_serialize_
- Serialize a type implementing `Serialize` trait using MessagePack format.
```rust
pub fn rmp_serialize<T>(val: &T) -> WasmResult<Vec<u8>>
```
### _rmp_deserialize_
- Serialize a type implementing `Deserialize` trait using MessagePack format.
```rust
pub fn rmp_deserialize<'a, T>(buf: &'a [u8]) -> WasmResult<T>
```

### _PackedValue_
- Value that has been already packed, thus it doesn't require further processing and shall be taken "as-is".
```rust
pub struct PackedValue(pub Vec<u8>);
```


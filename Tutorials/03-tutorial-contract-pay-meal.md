# Write the `pay-meal` smart contract from scratch

Let create a contract that allows to split the meal bill between all the diners
 - The information that we need to _initialize_ the contract is:
   - the `restaurateur` account
   - the list of `customers`
   - the `part` for each customer
   - the `asset` for the payment

 - Once the contract is initialized a customer must _apply_ to pay is share of the bill.

 - Once all the customers have paid their share the restaurateur could _close_ the contract, and collect the asset.

 - _Note:_ We use a [TAI compliant](../../Misc/Trinci-Application-Interface.md) asset
 - _Note:_ The following code is used as example, the full and properly working code can be found in the repository.

* _init_, _get_info_, _apply_, _close_ will become contract methods.


## `init` method

 - The `init` method in a smart contract usually should do some checks on the data (if needed), then store it in the `config` field of the account.
 - Can be called only by the account owner

### Writing and testing the `init` arguments
 - In the project we have just create open the file `types.rs` and add a structure for the init args:
   ```rust
   // Init Args
   #[derive(Serialize, Deserialize)]
   #[cfg_attr(test, derive(Debug, PartialEq, Clone))]
   pub struct InitArgs<'a> {
       pub restaurateur: &'a str, // is the merchant account
       pub asset: &'a str,        // is the asset account
       pub part: u64,             // is the the share for each diner
       pub customers: BTreeMap<&'a str, bool>, // the diners list
   }
   ```
 - Every contract method expects a binary argument that is [message pack](https://msgpack.org/) data.

 - Now we create a proper function in the `tests` section for the serialization/deserialization test of the init arguments.
   - create some account const field, like this:
     ```rust
     pub(crate) const RESTAURATEUR_ID: &str = "QmRestaurateurT8ijW7REd3KqN1kGBgYxFWsxajjguLky";
     pub(crate) const ASSET_ID: &str = "QmAssetT8ijsdfWsd7REf35d3KqN1kGBgYxFWsxajjguLk";
     pub(crate) const CUSTOMER1_ID: &str = "QmCustomer1-T8ijsdfWs35d3KqN1kGBgYxFWsxajjguLk";
     pub(crate) const CUSTOMER2_ID: &str = "QmCustomer2-T8ijsdfWs35d3KqN1kGBgYxFWsxajjguLk";
     pub(crate) const CUSTOMER3_ID: &str = "QmCustomer3-T8ijsdfWs35d3KqN1kGBgYxFWsxajjguLk";
     ```

   - create a function to create a `InitArgs` filled struct (will be useful for the method tests):
     ```rust
     pub(crate) fn create_init_args() -> InitArgs<'static> {
        let mut customers = BTreeMap::new();
        customers.insert(CUSTOMER1_ID, false);
        customers.insert(CUSTOMER2_ID, false);
        customers.insert(CUSTOMER3_ID, false);
        InitArgs {
            restaurateur: RESTAURATEUR_ID,
            asset: ASSET_ID,
            part: 30,
            customers,
        }
     }
     ```
   - add an empty constant that represent the hex serialized value of the init arguments, like this:
     ```rust
     const INIT_ARGS_HEX: &str = "";
     ```
   - write the tests for the serialization/deserialization:
     ```rust
     #[test]
     fn init_args_serialize() {
         let args = create_init_args();
 
         let buf = trinci_sdk::rmp_serialize(&args).unwrap();
 
         assert_eq!(hex::encode(&buf), INIT_ARGS_HEX);
     }
 
     #[test]
     fn init_args_deserialize() {
         let expected = create_init_args();
 
         let buf = hex::decode(INIT_ARGS_HEX).unwrap();
 
         let args: InitArgs = trinci_sdk::rmp_deserialize(&buf).unwrap();
 
         assert_eq!(args, expected);
     }
     ```
     - launch the test `init_args_serialize`, this obviously fails, but we can see how is the expected hex, we can try to deserialize it with ()[online tools] and replace in the `INIT_ARGS_HEX` constant.
       ```rust
       const INIT_ARGS_HEX: &str = "94d92e516d5265737461757261746575725438696a5737524564334b714e316b474267597846577378616a6a67754c6b79d92e516d41737365745438696a73646657736437524566333564334b714e316b474267597846577378616a6a67754c6b1e83d92e516d437573746f6d6572312d5438696a7364665773333564334b714e316b474267597846577378616a6a67754c6bc2d92e516d437573746f6d6572322d5438696a7364665773333564334b714e316b474267597846577378616a6a67754c6bc2d92e516d437573746f6d6572332d5438696a7364665773333564334b714e316b474267597846577378616a6a67754c6bc2";
       ```
     - launch all the tests.

 ### Writing and testing the `init` method
 - Open the file `lib.rs`
 - In Trinci rust contract a method has the following signature:
   ```rust
   fn function_name(ctx: AppContext, args: ArgumentsType) -> WasmResult<ReturnType>;
   ```
   where `ArgumentsType` and `ReturnType` are serializable structs.
 - In our case the `init` method shall take a `InitArgs` arguments type (we have just defined it in `types.rs`), and will return `nothing` or an `Error` with a `WasmResult` type that wraps a `std::result::Result` type, for now we write this:
   ```rust
   /// Init method
   fn init(_ctx: AppContext, _args: InitArgs) -> WasmResult<()> {
       Ok(())
   }
   ```
   - The rust compiler warning us that the `init` method is not used, this because we have not told the sdk to export this method. To do this we look for a row like this:
    ```rust
    trinci_sdk::app_export!(...);
    ```
    This is the macro that allows to make available our methods to the world, and simply add the `init` method inside the parenthesis, like this:
    ```rust
    trinci_sdk::app_export!(init);
    ```
  - Now the warning is disappeared but the `init` method still do nothing, to keep the contract simple we could implement the `init` method like this:
    ```rust
    /// Init method
    fn init(_ctx: AppContext, args: InitArgs) -> WasmResult<()> {
        trinci_sdk::store_account_data_mp!("config", &args)
    }
    ```
  - If we improve our contract we could add a check on the caller, and allow the contract initialization only at the account owner, putting the following at the start of the method code:
    ```rust
    if ctx.owner != ctx.caller {
        return Err(WasmError::new("not authorized"));
    }
    ```
  - To prevent not authorized asset withdraw we must use the TAI asset lock, like this:
    ```rust
    // Withdraw lock for the asset under escrow.
    trinci_sdk::asset_lock(args.asset, ctx.owner, trinci_sdk::tai::LockType::Withdraw)?;
    ```


  - Now we wrote a simple test function for the `init` method:
    ```rust
    #[test]
    fn test_init() {
        // we use an sdk function to create a context for the contract test
        let ctx = not_wasm::create_app_context(PAY_ID, PAY_ID);     

        // we use the function written previously to create the init arguments
        let args = create_init_args();

        // Associate a mock lock to the asset account
        not_wasm::set_contract_method(ASSET_ID, "lock", not_wasm::asset_lock);

        // This function simulate the contract method execution
        not_wasm::call_wrap(init, ctx, args.clone()).unwrap();

        // We make some checks on the account data to see
        // if the configuration as been properly written
        let buf = not_wasm::get_account_data(PAY_ID, "config");
        let data: InitArgs = rmp_deserialize(&buf).unwrap();

        assert_eq!(data, args);
    }
    ```
  - If we have added a owner checks we could write a test to check if the method fails if we call it from another account:
    ```rust
    #[test]
    fn test_init_not_authorized() {
        let ctx = not_wasm::create_app_context(PAY_ID, "Unknown");
        let args = create_init_args();

        // Associate a mock lock to the asset account
        not_wasm::set_contract_method(ASSET_ID, "lock", not_wasm::asset_lock);

        let err = not_wasm::call_wrap(init, ctx, args.clone()).unwrap_err();

        // Checks on the error
        assert_eq!(err.to_string(), "not authorized");
    }
    ```
  - To exercise we could write a more sophisticated init that take the total amount and calculate the customer part (the eventually round up could be see as the tip :) ) and store a struct different from the init args.

## `get_info` method
 - The `get_info` method allows to retrieve the information (config) of the contract
 - Can be called only by the account involved in the contract (the restaurateur or a customer)

### `get_info` arguments
 - In this case the `get_info` argument are `null`:
   ```json
   args: {}
   ```
 - We can use an SDK struct to represent a null arguments, `PackedValue`.

### `get_info` returns
 - The result is the `config` struct as serialized struct with names.
 - We realize that a `status` field would have been useful and then we go to insert it in the `InitArgs` struct, making all the necessary adjustments.
   ```rust
   {
      "restaurateur": account-id, // is the merchant account
      "asset": account-id,        // is the asset account
      "part": 123,                // is the the share for each diner
      "customers": {              // the diners list and the payment status
          "customer_1": true,
          "customer_2": false,
          "customer_3": false,
      }, 
      status: "open",             // contract status "open" or "closed"
   }
   ```

### Writing and testing the `get_info` method
 - This could be an example of the implementation of the `get_info` method:
 ```rust
  /// Get_Info method
 fn get_info(ctx: AppContext, _args: PackedValue) -> WasmResult<PackedValue> {
     // Load the contract configuration
     let buf = trinci_sdk::load_data("config");
     let config: InitArgs = match rmp_deserialize(&buf) {
         Ok(val) => val,
         Err(_) => return Err(WasmError::new("not initalized")),
     };
 
     // Checks on the authorization
     let mut auth_list = vec![config.restaurateur];
     config
         .customers
         .iter()
         .for_each(|(&customer, _)| auth_list.push(customer));
 
     match auth_list.iter().find(|&&elem| elem == ctx.caller) {
        Some(_) => {
            let buf = rmp_serialize_named(&config)?;
            Ok(PackedValue(buf))
        }
        None => Err(WasmError::new("not authorized")),
    }
 }
 ```

 - Let's write a test function for the `get_info` method:
 ```rust
 #[test]
 fn test_get_info() {
    // Prepare the environment
    // Prepare the context
    let ctx = not_wasm::create_app_context(PAY_ID, CUSTOMER1_ID);
  
    // Prepare the account data/config
    let data = create_init_args();
    let data = rmp_serialize(&data).unwrap();
    not_wasm::set_account_data(PAY_ID, "config", &data);
  
    let args = PackedValue::default();
  
    let res = not_wasm::call_wrap(get_info, ctx, args).unwrap();
  
    let data: Value = rmp_deserialize(&res.0).unwrap();
  
    // Checks on the contract info
    let restaurateur = data.get(&value!("restaurateur")).unwrap().as_str().unwrap();
  
    assert_eq!(restaurateur, RESTAURATEUR_ID);
 }
 ```
 - To exercise we could test the `get_info` with an not initialized contract and the `apply` from a unknown account

## `apply` method
The `apply` method performs the payment from a customer to the contract account for his bill share.

### `apply` arguments
 - In this case the `apply` argument are `null`:
   ```json
   args: {}
   ```


### Writing and testing the `apply` method
 - The `apply` method could be like this:
   ```rust   
   /// Apply method
   fn apply(ctx: AppContext, _args: PackedValue) -> WasmResult<()> {
       // Load the contract configuration
       let buf = trinci_sdk::load_data("config");
       let mut config: InitArgs = match rmp_deserialize(&buf) {
           Ok(val) => val,
           Err(_) => return Err(WasmError::new("not initalized")),
       };
   
       // Check if the caller is in the list and have already paid
       if let Some(val) = config.customers.get_mut(ctx.caller) {
           if !*val {
               // Make the payment
               trinci_sdk::asset_transfer(ctx.caller, ctx.owner, config.asset, config.part)
                   .map_err(|_| WasmError::new("transfer from caller failed"))?;
               *val = true;
           }
       } else {
           return Err(WasmError::new("not authorized"));
       };
   
       // Store the config and exit
       trinci_sdk::store_account_data_mp!("config", &config)?;
   
       Ok(())
   }
   ```
  - In this method will load the contract `config` and check if the caller is in the `customers` field and if has already paid his part
    - If the caller has not already paid a transfer will be performed from the caller to the contract (this could require a previously request for a `delegation` from the caller in the `Asset` account)
 - Now we are going to write some tests for the `apply` method:
   ```rust
   #[test]
   fn test_apply() {
       // Prepare the environment
       // Prepare the context
       let ctx = not_wasm::create_app_context(PAY_ID, CUSTOMER1_ID); 

       // Prepare the account data/config
       let data = create_init_args();
       let data = rmp_serialize(&data).unwrap();
       not_wasm::set_account_data(PAY_ID, "config", &data); 

       // Give the customer some asset
       not_wasm::set_account_asset_gen(CUSTOMER1_ID, ASSET_ID, Asset::new(100)); 

       // Associate a mock transfer to the asset account
       not_wasm::set_contract_method(ASSET_ID, "transfer", not_wasm::asset_transfer); 
       let args = PackedValue::default(); 
       
       not_wasm::call_wrap(apply, ctx, args).unwrap(); 

       // Checks on the contract account config
       let buf = not_wasm::get_account_data(PAY_ID, "config");
       let data: InitArgs = rmp_deserialize(&buf).unwrap();
       let customer1 = data.customers.get(CUSTOMER1_ID).unwrap(); 

       assert!(customer1); 

       // Checks on the contract asset
       let asset: Asset = not_wasm::get_account_asset_gen(PAY_ID, ASSET_ID); 

       assert_eq!(asset.units, 30);
   }
   ```

   - The `not_wasm_` module contains some useful functions that allow a user to build an environment to test the contract methods and performs some checks on the results:
     - `not_wasm::create_app_context(owner, caller) -> AppContext` - builds a context struct to pass to the methods that specifies the account owner and the account that call the method.
     - `not_wasm::set_account_data(account_id, key, data)` - allows to initialize an account data field with some content (binary), stored at `key`.
     - `not_wasm::get_account_data(account_id, key)` - allows to retrieve some data (as binary) from an account data field.
     - `not_wasm::set_account_asset_gen(account_id, asset_id, asset)` - allows to put some asset (must follow the TAI) in the `asset_id` field of the account `account_id`.
     - `not_wasm::get_account_asset_gen(account_id, asset_id)` - allows to retrieve the asset from the account specified.
     - `not_wasm::set_contract_method(account_id, method_name, method_func)` - allows to associate a mock function to the call of a method of an account contract in order to simulate a certain behavior.

  - To exercise we could test the `apply` with an not initialized contract and the `apply` from a unknown account
     
## `close` method
 - The `close` method allows the restaurateur to collect the asset from the contract
 - Can be called only by the restaurateur account
 - If every customer has paid the contract status will be set to closed, otherwise will be performed only the transfer of the current amount and the status will be leave "open".

### `close` arguments
 - In this case the `close` argument are `null`:
   ```json
   args: {}
   ```
 
### Writing and testing the `close` method
 - The `close` method looks like this:
 ```rust
 /// Close method
 fn close(ctx: AppContext, _args: PackedValue) -> WasmResult<()> {
     // Load the contract configuration
     let buf = trinci_sdk::load_data("config");
     let mut config: InitArgs = match rmp_deserialize(&buf) {
         Ok(val) => val,
         Err(_) => return Err(WasmError::new("not initialized")),
     };
 
     // Check if the caller is the restaurateur
     if ctx.caller != config.restaurateur {
         return Err(WasmError::new("not authorized"));
     }
 
     // Check if the contract is still opened
     if config.status != "open" {
         return Err(WasmError::new("contract closed"));
     }
 
     // Unlock the asset
     trinci_sdk::asset_lock(config.asset, ctx.owner, trinci_sdk::tai::LockType::None)?;
 
     // Get Balance
     let amount: u64 = trinci_sdk::asset_balance(config.asset)?;
 
     // Perform the Transfer
     trinci_sdk::asset_transfer(ctx.owner, config.restaurateur, config.asset, amount)?;
 
     // Lock again the asset
     trinci_sdk::asset_lock(config.asset, ctx.owner, trinci_sdk::tai::LockType::Withdraw)?;
 
     // If all the customers have paid set the status to close
     if !config.customers.values().any(|&val| !val) {
         //  All the customers have paid
         config.status = "close";
         trinci_sdk::store_account_data_mp!("config", &config)?;
     }
 
     Ok(())
 }
 ```
 - Pay attention to the asset lock handling!

# Compiling the contract
 - To compile the contract we go in the '<repo>/app-rs' directory and launch the command:
   ```bash 
   $ ./build_wasm.sh
   ``` 
   if we want to build the contract with our rust toolchain
   or
   ```bash 
   $ ./build_wasm_docker.sh
   ```
   if we want to build the contract with a toolchain in a pre-build docker image.
 - The previously commands will build each contract in the directory and copy the `.wasm` files in the `<repo>/registry` directory

# Integration Test
 - Now that we have made a contract by scratch and tested its methods we need to perform a more accurate test using the real contract that we have mocked in the unit tests (in this case just the asset contract)

## The Environment
 - The integration test project is inside the Trinci project, in our repository we could find a project under  `<repo>/integration` that uses the trinci modules to perform integration test.
 - We need to create a new `pay_meal.rs` file (we could created it from scratch or we could just copy and modify another contract file)
 - The core of an integration test is the test method itself:
   ```rust
   #[test]
   fn pay_meal_test() {
       // Instance the application.
       let mut app = TestApp::default();
   
       // Create and execute transactions.
       let txs = create_txs();
       let rxs = app.exec_txs(txs);
       check_rxs(rxs);

       // Performs account checks
       ...
   }
   ```
    - In the first section we initialize the environment
    - Then we use a function to create the transactions to execute
    - Then we execute the transactions
    - Then we check the answer 
    - Then we perform other checks on the account

## Build the Various Scenarios 
 - When we build an integration test we need to think it like a scenario where our contract is the main actor. 
   In our case we could think something like this:
    - We have 3 people (Luigi, Bruno, Marco) that had dinner at Mario's Pizza
    - They have spent 90 units of asset in total (30 each)
    - Marco get the bill information
    - Luigi pays his bill
    - Luigi tries to pay again his bill. This shall fail
    - Piero (not an actual customer) tries to pay in this contract. This shall fail.
    - Piero get the contract information. This shall fail.
    - Marco try to close the contract. This shall fail.
    - Mario (the restaurateur) try to close the contract.
    - Marco and Bruno pay their bill.
    - Mario (the restaurateur) try to close the contract.
    - Mario get the contract information.

## Build the Transactions creator
 - Lets build the transaction for the previously scenario.
 - We use a function like this (see the full code):
 ```rust
 fn create_txs() -> Vec<Transaction> {
     let contract_info = ACCOUNTS_INFO.get(PAY_ALIAS).unwrap();
     let restaurateur_info = ACCOUNTS_INFO.get(RESTAURATEUR_ALIAS).unwrap();
     let marco_info = ACCOUNTS_INFO.get(MARCO_ALIAS).unwrap();
     let luigi_info = ACCOUNTS_INFO.get(LUIGI_ALIAS).unwrap();
     let bruno_info = ACCOUNTS_INFO.get(BRUNO_ALIAS).unwrap();
     let piero_info = ACCOUNTS_INFO.get(PIERO_ALIAS).unwrap();
     let asset_info = ACCOUNTS_INFO.get(ASSET_ALIAS).unwrap();
 
     vec![
         // 0. Initialize src asset
         asset_init_tx(asset_info, ASSET_ALIAS),
         // 1. Mint some units in customers account.
         asset_mint_tx(asset_info, marco_info, 100),
         // 2. Mint some units in customers account.
         asset_mint_tx(asset_info, luigi_info, 100),
         // 3. Mint some units in customers account.
         asset_mint_tx(asset_info, bruno_info, 100),
         // 4. Mint some units in customers account.
         asset_mint_tx(asset_info, piero_info, 100),
         // 5. Initialize contract account.
         contract_init_tx(
             contract_info,
             restaurateur_info,
             asset_info,
             marco_info,
             luigi_info,
             bruno_info,
             30,
         ),
         // 6. Marco get the contract info
         contract_get_info_tx(contract_info, marco_info),
         // 7. Luigi add delegation to pay the bill
         asset_add_delegation_tx(asset_info, luigi_info, contract_info, 30),
         // 8. Luigi pays his bill
         contract_apply_tx(contract_info, luigi_info),
         // 9. Piero tries to pay. This shall fail.
         contract_apply_tx(contract_info, piero_info),
         // 10. Piero tries to get contract information. This shall fail.
         contract_get_info_tx(contract_info, piero_info),
         // 11. Marco tries to close the contract. This shall fail.
         contract_close_tx(contract_info, marco_info),
         // 12. Mario (the restaurateur) tries to close the contract.
         contract_close_tx(contract_info, restaurateur_info),
         // 13. Bruno add delegation to pay the bill
         asset_add_delegation_tx(asset_info, luigi_info, contract_info, 30),
         // 14. Bruno pays his bill
         contract_apply_tx(contract_info, luigi_info),
         // 15. Marco add delegation to pay the bill
         asset_add_delegation_tx(asset_info, marco_info, contract_info, 30),
         // 16. Marco pays his bill
         contract_apply_tx(contract_info, marco_info),
         // 17. Mario (the restaurateur) tries to close the contract.
         contract_close_tx(contract_info, restaurateur_info),
         // 18. Mario get the contract information
         contract_get_info_tx(contract_info, restaurateur_info),
     ]
 }
 ```
 - There are some function that are relative at other contract that are relate to the contracts we are going to interact with (in this case just the `asset` with `init` and `mint`)

## Build the Check Receipts 
 - The method that checks the receipts will check if the transaction has given the expected result (null, or some data) or an error (in this case we check the expected error code):
   ```rust
   fn check_rxs(rxs: Vec<Receipt>) {
       // 0. Initialize src asset
       assert!(rxs[0].success);
       // 1. Mint some units in customers account.
       assert!(rxs[1].success);
       // 2. Mint some units in customers account.
       assert!(rxs[2].success);
       // 3. Mint some units in customers account.
       assert!(rxs[3].success);
       // 4. Mint some units in customers account.
       assert!(rxs[4].success);
       // 5. Initialize contract account.
       assert!(rxs[5].success);
       // 6. Marco get the contract info
       assert!(rxs[6].success);
       // Checks on the config
       let config: Value = rmp_deserialize(&rxs[6].returns).unwrap();
       let status = config.get(&value!("status")).unwrap().as_str().unwrap();
       assert_eq!(status, "open");
       // 7. Luigi add delegation to pay the bill
       assert!(rxs[7].success);
       // 8. Luigi pays his bill
       assert!(rxs[8].success);
       // 9. Piero tries to pay. This shall fail.
       assert!(!rxs[9].success);
       assert_eq!(
           String::from_utf8_lossy(&rxs[9].returns),
           "smart contract fault: not authorized"
       );
       // 10. Piero tries to get contract information. This shall fail.
       assert!(!rxs[10].success);
       assert_eq!(
           String::from_utf8_lossy(&rxs[9].returns),
           "smart contract fault: not authorized"
       );
       // 11. Marco tries to close the contract. This shall fail.
       assert!(!rxs[11].success);
       assert_eq!(
           String::from_utf8_lossy(&rxs[9].returns),
           "smart contract fault: not authorized"
       );
       // 12. Mario (the restaurateur) tries to close the contract.
       assert!(rxs[12].success);
       // 13. Bruno add delegation to pay the bill
       assert!(rxs[13].success);
       // 14. Bruno pays his bill
       assert!(rxs[14].success);
       // 15. Marco add delegation to pay the bill
       assert!(rxs[15].success);
       // 16. Marco pays his bill
       assert!(rxs[16].success);
       // 17. Mario (the restaurateur) tries to close the contract.
       assert!(rxs[17].success);
       // 18. Mario get the contract information
       assert!(rxs[18].success);
       // Checks on the config
       let config: Value = rmp_deserialize(&rxs[18].returns).unwrap();
       let status = config.get(&value!("status")).unwrap().as_str().unwrap();
       assert_eq!(status, "close");
   }
   ```

 - _Note:_ Pay attention adding transaction in the middle of the above `create_txs` method, because this implies that the receipts `index` must be update in each following `assert`!

 - Last thing we add some blockchain checks:
   ```rust
   ...
   // Blockchain check.
   let asset_info = ACCOUNTS_INFO.get(ASSET_ALIAS).unwrap();
   let contract_info = ACCOUNTS_INFO.get(PAY_ALIAS).unwrap();
   let restaurateur_info = ACCOUNTS_INFO.get(RESTAURATEUR_ALIAS).unwrap();
   let marco_info = ACCOUNTS_INFO.get(MARCO_ALIAS).unwrap();
   let luigi_info = ACCOUNTS_INFO.get(LUIGI_ALIAS).unwrap();
   let bruno_info = ACCOUNTS_INFO.get(BRUNO_ALIAS).unwrap();
   let piero_info = ACCOUNTS_INFO.get(PIERO_ALIAS).unwrap();
 
   let contract_account = app.account(&contract_info.id).unwrap();
   let contract_asset: Asset =
       serialize::rmp_deserialize(&contract_account.load_asset(&asset_info.id)).unwrap();
   assert_eq!(contract_asset.units, 0);
 
   let restaurateur_account = app.account(&restaurateur_info.id).unwrap();
   let restaurateur_asset: Asset =
       serialize::rmp_deserialize(&restaurateur_account.load_asset(&asset_info.id)).unwrap();
   assert_eq!(restaurateur_asset.units, 90);
 
   let bruno_account = app.account(&bruno_info.id).unwrap();
   let bruno_asset: Asset =
       serialize::rmp_deserialize(&bruno_account.load_asset(&asset_info.id)).unwrap();
   assert_eq!(bruno_asset.units, 70);
 
   let marco_account = app.account(&marco_info.id).unwrap();
   let marco_asset: Asset =
       serialize::rmp_deserialize(&marco_account.load_asset(&asset_info.id)).unwrap();
   assert_eq!(marco_asset.units, 70);
 
   let luigi_account = app.account(&luigi_info.id).unwrap();
   let luigi_asset: Asset =
       serialize::rmp_deserialize(&luigi_account.load_asset(&asset_info.id)).unwrap();
   assert_eq!(luigi_asset.units, 70);
 
   let piero_account = app.account(&piero_info.id).unwrap();
   let piero_asset: Asset =
       serialize::rmp_deserialize(&piero_account.load_asset(&asset_info.id)).unwrap();
   assert_eq!(piero_asset.units, 100);
   ...

   ```

# `DRAFT`

# Installing Rust
   - Linux or Mac OS: 
     ```bash 
     $ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
     ```
     - When shown choose the default settings
     - require git, curl and build-essential:
       ```bash
       # apt install git curl build-essential
       ```
   - Windows: see the [rust official site](https://forge.rust-lang.org/infra/other-installation-methods.html)

# Download, compile and Run Trinci Node
   - The Trinci Node and Rust SDK source code can be found on github: [Affidaty Blockchain](https://github.com/affidaty-blockchain)

   - In order to build trinci we need to install the following dependencies: 

     ```bash
     clang libclang-dev protobuf-compiler
     ```

   - We can download the trinci node repository and then build the node:
   
     ```bash
     $ git clone https://github.com/affidaty-blockchain/trinci-node.git
     ```
   
   - Then we can build the executables.
     
     To download the needed Rust crates from crates.io
     ```bash
     $ cargo check
     ````
     To build the executables:
     ```bash
     $ cargo build
     ```
     To run an executable:
     ```bash
     $ cargo run --bin <bin-executable> -- <arguments>
     ```
     - Example:
       ```bash
       cargo run --bin trinci-node -- --validator --network skynet
       ```
       This command launch a node as validator in the `skynet` network.


# Download Trinci Smart Contract Development Environment
   - The Trinci Smart Contract Development Environment can be found on github: [Trinci SmartContracts](https://github.com/affidaty-blockchain/trinci-smartcontracts)

   ```bash
   $ git clone https://github.com/affidaty-blockchain/trinci-smartcontracts.git
   ```

   - This crate depends on the [trinci-sdk](https://github.com/affidaty-blockchain/trinci-sdk-rust) crate. 
   - The Trinci Rust SDK documentation can be found [here](./02-tutorial-Rust-SDK).


## Create a new contract
  - Use the script `create_new_contract.sh` in the directory <repo>/app-rs/:
    ```bash
    $ ./create_new_contract.sh
    ```
    then insert the new contract name.
    A project with the name provided will be created.
     - Note requires the [`cargo generate`](https://crates.io/crates/cargo-generate) crate.
  
  
  ### Launch the same cargo command for all the contracts:
  ```bash
  $ ./cargo_broadcast.sh <COMMAND>
  ```
  
  Example used to test all the contracts: 
  ```bash
  $ ./cargo_broadcast.sh test
  ```
  
  ### Compile all the contracts with the rust installed on our computer
  ```
  $ ./build_wasm.sh
  ```
   - Note: require `rust` with target `wasm32-unknown-unknown`
  
  ### Compile all the contracts with a docker image
  ```
  $ ./build_wasm_docker.sh
  ```
   - Note: requires `docker`
  
  
  ## `registry` directory 
   - The `registry` directory contains the .wasm contracts
  
  
  ## `integration` directory 
   - The `integration` directory is a rust module that provide an enviroment for testing

## Register a new smart contract
   - There are two ways to register a smart contract in Trinci Blockchain
     - Through a transaction to the `Service account` with arguments:
       ```json
       args: {
           "name": string,          // contract name
           "version": string,       // contract version
           "description": string,   // contract description
           "url": string,           // contract web site
           "bin": binary,           // contract binary
       }
       ```      
     - With the Trinci CLI:
       ```bash
       $ trinci-cli 
             --host <trinci_host> \
             --port 80 <trinci_port> \
             --path <trinci_path>/api/v1 \
             --network <trinci_network>
       ```
       - Enter in the `cheats` section:
       ```bash
       Enter 'help' to show available commands
       > cheats
       ```
       - Launch the `register` command:
       ```bash
       >>> register
       ```
       - Fill with the needed information:
       ```bash
         Service account: Qm...xyz                          # the service account
         Service contract (optional multihash hex string):  # just press `enter`
         New contract name: My Cool Contract                # the contract name
         New contract version: 0.2.1                        # contract version
         New contract description: This is my cool contract 
         New contract url: www.cool-contract.org
         New contract filename: /home/contracts/my_cool_contract.wasm # contract file in our filesystem
       ```



# Build a contract from scratch
 - In [this section](./03-tutorial-contract-pay-meal) will are going to build a smart contract from zero

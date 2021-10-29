Assets
======

_Asset_ is the currency circulating on a blockchain.
 - An asset could be:
   - Created and added to the blockchain
   - Minted, that means coined out of nothing
   - Transferred from an account to another (the transfer could be handled from a [custom smart contract](Smart-contracts/Basic-Asset-smart-contract.md) created specifically for the asset)
 - The _asset_ is stored on the `data` field of the _service_ account in a format like this:
   ```json
	{
	   "assets": {
		  asset-account-id: {            // Asset account id
			 "name": string,             // Asset name					 
			 "creator": account-id,      // Caller account-id
			 "url": integer,       		 // Web site of the asset
			 "contract": bytes           // Hash of the contract that handle the asset
		  },
	   }
	}
   ```

 - An Asset account contains in its data section all the asset features static and dynamic.
- The related contract defines the asset behaviour.
- The Asset features are specific to the type of asset and only the related smart contract enters into the content.
- Assets can have lists of features completely different (even with different names to mean the same thing).
- An asset smart contract could be used by more than one asset
  Example:
  - Let consider two "simple" asset like `BTC` and `EUR`, they have the same behaviour.
  - So they can be handled by the same `basic_asset` smart contract.
  - But they have different configuration.
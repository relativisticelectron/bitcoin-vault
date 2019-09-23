# Bitcoin Vault
An implementation of a bitcoin vault with currently available op_codes on bitcoin testnet.

A bitcoin vault are 2 bitcoin addresses (A and B). You store bitcoins in address A long-tern. Address A can only spend to B. Such a transaction ( A --> B) could be initiated either by you or a thief.  The crucial point is that the thief has to wait a certain time, e.g. 2 weeks, before he can initiate a spend from B to his own address. However you noticed the transaction A --> B on the blockchain and initiate an immediate transaction B --> your address with your secret seed E, therefore stopping the theft. 

**Current status:** The ideal version of address A (bitcoin vault), with a spend restriction, cannot be realized with current Bitcoin script.
**Workaround (this script):** One can realize bitcoin vaults however with partial signed transactions and deleting of private keys (yes, really deleting).  The attached script realizes exactly this workaround on bitcoin testnet. 
**Future outlook:** Though it is possible that these covenant op_codes  for real bitcoin vaults will be included in the future, it is not certain because: If such op_codes would be implemented one could have recursive restrictions on spending, effectively locking the bitcoins forever (except for fees). Prior to implementation one need to carefully evaluate (usage) risks of these op_codes. Until then this workaround method could be a viable path to limit bitcoin thefts in an effective manor.

**Possible implementations:** There are at least 2 possible ways of enforcing the timelock in the transactions.

 1. Property of Address B: Using  [OP_CHECKSEQUENCEVERIFY](https://masteringbitcoin.neocities.org/#_timelocks) (OP_CSV)  during creation of address B one can enforce that spending of B requires the coins stayed at least, e.g., 2 weeks on B.
 2. Transaction B --> your address: Using  [nSequence](https://masteringbitcoin.neocities.org/#_timelocks) in the transaction itself one can enforce that the coins stayed at least, e.g., 2 weeks on B.

While option 1 is more elegant, we implement here option 2 due to simplicity.

## High level documentation
A documentation of the address and transaction setup can be found in [Bitcoin vault.pdf](Bitcoin%20vault.pdf).  This explains also the short names used in the code for private keys and wallets.

##  Dependencies
In Ubuntu simply do:
```sh
sudo apt install libgmp-dev
pip install bitcoinlib
```
## Usage

### Debug mode
Running in debug mode:  Simply run the entire script
* This mode will not delete any private keys
* It will give you additionally all completely signed transactions
* You can broadcast all 3 fully signed transactions here:   https://tbtc.bitaps.com/broadcast  and check on   https://live.blockcypher.com
	1. Source wallet --> A/vault
	2. A/vault --> B/release
	3. B/release --> final wallet/source wallet  (this one has a timelock)!
		* You could use the emergency key E and normal key R2 to initiate a spend transaction immediately

### Production mode
* This mode will delete all non-relevant private keys
* This will give you the signed transaction which you can broadcast here:   https://tbtc.bitaps.com/broadcast   and check on   https://live.blockcypher.com
	1. Source wallet --> A/vault
* This will only give you the partially signed transaction
	2. A/vault --> B/release
	3. B/release --> final wallet/source wallet  (this one has a timelock)!
		* You could use the emergency key E and normal key R2 to initiate a spend transaction immediately
* **Storing of keys**
	* The emergency seed E should be kept on paper and secret (it's the testnet, but anyway) 
	* Store the data.pickle file on the computer  (with backups). It will allow a normal withdraw of funds to your final wallet and contains
		* Seed of R1
		* Seed of R2
		* Partially signed transactions 
			2. A/vault --> B/release
			3. B/release --> final wallet/source wallet  (this one has a timelock)!

#### Releasing the funds
One can release the funds the following way:
1. Sign transaction A/vault --> B/release  with the key generated by the seed R1
2. Broadcast it
3. Wait the locktime
4. Sign transaction B/release --> final wallet/source with the key generated by the seed R2
5. Broadcast it

#### Attack scenarios
##### Winning scenarios
* An attacker has access to:
	* R2 and
	* the partially signed transaction B/release --> final wallet/source
* An attacker has access to:
	* fully signed transaction A/vault --> B/release
	* fully signed transaction B/release --> final wallet/source 

##### Losing scenarios / No advantage scenarios
* An attacker has access to:
	* seed E
	* seed R2
* An attacker has any of the seeds that you deleted and various other transactions or seeds
* An attacker has access to:
	* seed R2 and
	* the partially signed transaction B/release --> final wallet/source and
	* the private keys to final wallet/source


## Reading material and tools


* On-Chain defense: [https://www.youtube.com/watch?v=diNxp3ZTquo](https://www.youtube.com/watch?v=diNxp3ZTquo) 
* A paper on bitcoin covenants  [http://fc16.ifca.ai/bitcoin/papers/MES16.pdf](http://fc16.ifca.ai/bitcoin/papers/MES16.pdf)
* An explanation of timelock op_codes: https://www.bgp4.com/2018/10/12/bitcoins-time-locks/
* Another explanation of timelock op_codes: https://masteringbitcoin.neocities.org/#_timelocks


### Testnet tools
*  Broadcast raw transations: https://tbtc.bitaps.com/broadcast
*  Decode raw transactions: https://live.blockcypher.com/btc-testnet/decodetx/
*  A testnet faucet: https://testnet-faucet.mempool.co/

## Notes 
* Bitcoin vaults, without extending the Bitcoin script, have the downside that you have to hold the private keys V1 and V2 for the time it takes to receive the funding transaction. The steps to fund the vault are:
	1. generate all private keys/seeds
	2. Send the public vault address to the  funding wallet
	3. Construct transaction funding wallet --> vault
	4. Construct all other partial transactions 
	5. Delete keys/seeds  V1 and V2
* The vault address **MUST NOT** be sent to again, because it is unspendable after deleting V1

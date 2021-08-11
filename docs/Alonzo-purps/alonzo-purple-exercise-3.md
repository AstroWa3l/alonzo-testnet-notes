# Part 1
***Create second wallet and send test ada to it.***
```bash
cardano-cli address key-gen \
--verification-key-file payment2.vkey \
--signing-key-file payment2.skey
```

```bash
cardano-cli stake-address key-gen \
--verification-key-file stake2.vkey \
--signing-key-file stake2.skey
```

```bash
cardano-cli address build \
--payment-verification-key-file payment2.vkey \
--stake-verification-key-file stake2.vkey \
--out-file payment2.addr \
-testnet-magic 8
```
## Get protocol parameters
```bash
cardano-cli query protocol-parameters   --alonzo-purple   --out-file pparams.json
```
## Get the transaction hash and index of the UTXO to spend
```bash
cardano-cli query utxo --testnet-magic 8 --address $(cat payment.addr)
```

## Build the transaction

```bash
cardano-cli transaction build-raw \
--tx-in 134cfaf19595ce23dfc6664a7bdba06a371f5606988413397195d731fed6c5cc#0 \
--tx-out $(cat payment2.addr)+0 \
--tx-out $(cat payment.addr)+0 \
--fee 0 \
--out-file tx.draft
```

```bash
cardano-cli transaction calculate-min-fee \
--tx-body-file tx.raw \
--tx-in-count 1 \
--tx-out-count 2 \
--witness-count 1 \
--testnet-magic 8 \
--protocol-params-file pparams.json
```
`177513 Lovelace`

```bash
cardano-cli transaction build-raw \
--alonzo-era \
--fee 177513 \
--tx-in 134cfaf19595ce23dfc6664a7bdba06a371f5606988413397195d731fed6c5cc#0 \
--tx-out $(cat payment2.addr)+25000000000 \
--tx-out $(cat payment.addr)+974999822487 \
--protocol-params-file pparams.json \
--out-file tx.raw
```

```bash
cardano-cli transaction sign \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--testnet-magic 8 \
--out-file tx.sign
```

```bash
cardano-cli transaction submit --tx-file tx.sign --testnet-magic 8
```

# Part 2
***Running basic Plutus scripts and getting use to the command line***

## Generate random number and use CLI to hash your random number

```bash
cardano-cli transaction hash-script-data --script-data-value $(cat random_datum.txt)
```
- Save the value hash to `random_datum_hash.txt`

## Generate script address

```bash
cardano-cli address build --payment-script-file AlwaysSucceeds.plutus --testnet-magic 8 --out-file script.addr
```
## Generate protocol-parameters file
```bash
cardano-cli query protocol-parameters --testnet-magic 8 > pparams.json
```

## Send funds to the script address, include the datum hash
```bash
cardano-cli transaction build-raw \
--alonzo-era \
--tx-in 0a2acda76afb6d4b800a6d6fe9efb80b4a5d396c67be2668f4055196da046d2c#0 \
--tx-out $(cat script.addr)+10000000000 \
--tx-out-datum-hash $(cat random_datum_hash.txt) \
--tx-out $(cat payment2.addr)+14999000000 \
--fee 1000000 \
--protocol-params-file pparams.json \
--out-file tx.raw
```

```bash
cardano-cli transaction sign --tx-body-file tx.raw --signing-key-file payment2.skey --testnet-magic 8 --out-file tx.sign
```

```bash
cardano-cli transaction submit --testnet-magic 8 --tx-file tx.sign
```

## Query the script address balance
```bash
cardano-cli query utxo --testnet-magic 8 --address $(cat script.addr)
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
32eb8b686f7f62488c540e35ab94f7e11548f71c6e2af1291f623122131ceac4     1        989999000000 lovelace + TxOutDatumHashNone
4338cb66573353c368a716ed35c46d7f19d0474a156979d88b0678085e138282     0        99811570 lovelace + TxOutDatumHashNone
435eb9c07520bf0aa93d0ff69083b1d4e4ab72884c494f99eed0a3ae7bdc883c     0        49813066 lovelace + TxOutDatumHashNone
481a01c36c6f2dcb0f031e1e6d1a255a8773138f7fae11596a7ff53f1f5c5c54     0        8888880 lovelace + TxOutDatumHashNone
5925dc51db2fe7d168db02dc9e9fff61c4eb93e65dadaa22592aa0902ae1c998     0        999813154 lovelace + TxOutDatumHashNone
6bafbd83b11f0d70a980c8cbaa1422ffd943db9262841600e242a0ef4bfdf312     0        1645849 lovelace + TxOutDatumHashNone
98cf51f0e7cf191e9accc3825cfb92b355f7fa7662d4adfef71eb809e4a70161     1        998999000000 lovelace + TxOutDatumHashNone
a2a2b20eccac7fb5e4a0172564d0b2243ac56e1bc3652ed155610ee63802a045     1        3456789 lovelace + TxOutDatumHashNone
fedbfc79d12d27c891dca82ab3110cf3077e6536b9049949ffdef199698971cf     0        10000000000 lovelace + TxOutDatumHash ScriptDataInAlonzoEra "6d683f9bd0c3b874cdf3da1793b5eb0ea73a074d3e4b66bc62279b09d387fa8d"
```
- Note: We can use the tx id and/or the datum hash to identify "our" UTxO at the script, which is the last on in this case. The script address has a UTxO that can ONLY be spent when providing the VALUE that hashes to, for this case we would need the hash: `6d683f9bd0c3b874cdf3da1793b5eb0ea73a074d3e4b66bc62279b09d387fa8d`. 

# Part 3 (Not complete)
***Spending the funds from the script address***
```bash
COLLATERAL=3064e641ab188863eb3ec7755d1c06ba4c95b3db5575a87998c95c7c8053430a#1
ADDR3=ADDR3=fedbfc79d12d27c891dca82ab3110cf3077e6536b9049949ffdef199698971cf#0
MAGIC=8
```
```bash
 cardano-cli transaction build \
 --alonzo-era \
 --tx-in ${ADDR3} \
 --tx-in-script-file AlwaysSucceeds.plutus \
 --tx-in-datum-value $(cat random_datum.txt) \
 --tx-in-redeemer-value $(cat random_datum.txt) \
 --tx-in-collateral ${COLLATERAL} \
 --change-address $(cat payment2.addr) \
 --protocol-params-file pparams.json \
 --testnet-magic ${MAGIC} \
 --out-file tx.raw
 ```
cardano-cli transaction sign --tx-body-file tx.raw --signing-key-file collateral_payment.skey --testnet-magic ${MAGIC} --out-file tx.sign
cardano-cli transaction submit --testnet-magic ${MAGIC} --tx-file tx.sign

export MAGIC="--testnet-magic 8"
mkdir -p ~/alonzo-testnet/files/senderWallet
cd alonzo-testnet/files/senderWallet

## Create sender wallet

cardano-cli address key-gen \
--verification-key-file payment.vkey \
--signing-key-file payment.skey

cardano-cli stake-address key-gen \
--verification-key-file stake.vkey \
--signing-key-file stake.skey

cardano-cli address build \
--payment-verification-key-file payment.vkey \
--stake-verification-key-file stake.vkey \
--out-file payment.addr \
--testnet-magic 8

cat payment.addr
addr_test1qzsdrv4juj5hmc3dqeq9wc7kpjwmu69zwjz9j29m3l88m26hdfyr7r54f67e7c9de70ld7gzfwmhqpudvkyyvkse2gmq0cvrdj
export ADDRESS1=addr_test1qzsdrv4juj5hmc3dqeq9wc7kpjwmu69zwjz9j29m3l88m26hdfyr7r54f67e7c9de70ld7gzfwmhqpudvkyyvkse2gmq0cvrdj

cardano-cli stake-address build \
--stake-verification-key-file stake.vkey \
--out-file stake.addr \
--testnet-magic 8

cat stake.addr
stake_test1uptk5jplp625a0vlvzkul8lklypyhdmsq7xktzzxtgv4ydsk4g6es

##
mkdir -p ~/alonzo-testnet/files/receiverWallet
cd ~/alonzo-testnet/files/receiverWallet


## Create receiving wallet
```bash
cardano-cli address key-gen \
--verification-key-file payment.vkey \
--signing-key-file payment.skey
```

```bash
cardano-cli stake-address key-gen \
--verification-key-file stake.vkey \
--signing-key-file stake.skey
```

```bash
cardano-cli address build \
--payment-verification-key-file payment.vkey \
--stake-verification-key-file stake.vkey \
--out-file payment.addr \
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
cardano-cli transaction build \
--alonzo-era \
--tx-in 38f5b563616650fb61d83f22b2c29542eb3e5fed28dc3b6db8250cbb06434bc5#0 \
--tx-out $(cat ~/alonzo-testnet/files/receiverWallet/payment.addr)+25000000000 \
--change-address $(cat ~/alonzo-testnet/files/senderWallet/payment.addr) \
--testnet-magic 8 \
--out-file tx.raw
```

## sign the transaction

```bash
cardano-cli transaction sign \
--testnet-magic 8 \
--signing-key-file ~/alonzo-testnet/files/senderWallet/payment.skey \
--tx-body-file tx.raw \
--out-file tx.sign
```

## submit transaction

```bash
cardano-cli transaction submit --testnet-magic 8  --tx-file tx.sign
```

## Check the receiving address to make sure funds arrived

```bash
cardano-cli query utxo --testnet-magic 8 --address $(cat ~/alonzo-testnet/files/receiverWallet/payment.addr)
```

## Use the CLI to hash your random number and save the hash to `random_datum_hash.txt`

cardano-cli transaction hash-script-data --script-data-value $(cat random_datum.txt) > random_datum_hash.txt


## Generate the script address

cardano-cli address build --payment-script-file AlwaysSucceeds.plutus ${MAGIC} --out-file script.addr

## Generate protocol-parameters file

cardano-cli query protocol-parameters ${MAGIC} > pparams.json

## Send funds to the script address, we must include the datum hash

### Build the transaction using `transaction build` (recommended)
      
cardano-cli transaction build \
--alonzo-era \
--tx-in e05b2a322a2b0f2b0910c0543e7b804a0a40d864add13c07841287e41b13053b#1 \
--tx-out $(cat script.addr)+10000000000 \
--tx-out-datum-hash $(cat random_datum_hash.txt) \
--change-address $(cat ~/alonzo-testnet/files/receiverWallet/payment.addr) \
${MAGIC} \
--protocol-params-file pparams.json \
--out-file tx.raw

### So now we can sign and submit the transaction

cardano-cli transaction sign --tx-body-file tx.raw --signing-key-file ~/alonzo-testnet/files/receiverWallet/payment.skey ${MAGIC} --out-file tx.sign

cardano-cli transaction submit ${MAGIC} --tx-file tx.sign

### Query the script address balance

We can use the tx id and/or the datum hash to identify "our" UTxO at the script. The last one, in this case:

    cardano-cli query utxo ${MAGIC} --address $(cat script.addr)

## Build tx

 cardano-cli transaction build \
 --alonzo-era \
 --tx-in cdfae3971c135f61a53209a1282c924fc6f1bcafaf2296f4a3b475019e419e82#1 \
 --tx-in-script-file AlwaysSucceeds.plutus \
 --tx-in-datum-value $(cat random_datum.txt) \
 --tx-in-redeemer-value $(cat random_datum.txt) \
 --tx-in-collateral 9aad0488b78f7116e11af376a6813bb91017574fea31f985bd494af24b42830f#0 \
 --change-address $(cat ~/alonzo-testnet/files/receiverWallet/payment.addr) \
 --protocol-params-file pparams.json \
 ${MAGIC} \
 --out-file tx.raw

## Sign the transaction

cardano-cli transaction sign --tx-body-file tx.raw --signing-key-file ~/alonzo-testnet/files/receiverWallet/payment.skey ${MAGIC} --out-file tx.sign
cardano-cli transaction submit ${MAGIC} --tx-file tx.sign

## Get the transaction hash and index of the UTXO to spend
```bash
cardano-cli query utxo --testnet-magic 8 --address $(cat payment.addr)
```

## Build the transaction


cardano-cli transaction build \
--alonzo-era \
--tx-in 37b02dc4037fb68cc7add50cf47e563066e0c36445180d17643fda717bac87bf#0 \
--tx-in 73972694d5d63db276734a66818124dacd322e93407e47e4a085a3534d3c60d5#1 \
--tx-out $(cat payment.addr)+1000000000000 \
--change-address $(cat paymentbromel.addr) \
--testnet-magic 8 \
--out-file tx.raw

# Sign the transaction

cardano-cli transaction sign \
--testnet-magic 8 \
--signing-key-file payment.skey \
--tx-body-file tx.raw \
--out-file tx.sign


## submit transaction

```bash
cardano-cli transaction submit --testnet-magic 8  --tx-file tx.sign
```

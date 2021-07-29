## Part 1 Create a new set of keys and address

1. Create a new set of keys and address.

```bash
cardano-cli address key-gen \
--verification-key-file payment2.vkey \
--signing-key-file payment2.skey

cardano-cli stake-address key-gen \
--verification-key-file stake2.vkey \
--signing-key-file stake2.skey

cardano-cli address build \
--payment-verification-key-file payment2.vkey \
--stake-verification-key-file stake2.vkey \
--out-file payment2.addr \
--testnet-magic 7
```
- Querying the address:
```bash
cardano-cli query utxo --testnet-magic 7 --address $(cat payment.addr)
```
                         TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
55ba9e9542b8962a6689f69242158d665bde7e610e4e5af9ebe9e080ad2024a2     1        10000000 lovelace + TxOutDatumHashNone
7b4956b103d47908318ee92aa0790ff4b36fe7940991f0be350c9085fc4da175     1        100000000000 lovelace + TxOutDatumHashNone

### Send tokens between the funded wallet and the new address

1. Create the transaction.

```bash
cardano-cli transaction build-raw \
--mary-era \
--fee 200000 \
--tx-in f3106e23dc9f794e6a6162e26f5aa9a107dad6fffd0204f534e792c95bf2048f#1 \
--tx-out addr_test1qqtc6l8g0kt2hvtd6xsesamrxe5pknacr5xw5e023x0c2k5u8clq4clmksmxvtwp388876g5skfyzzyjhtwyvylt3lzq3dxkr4+25000000000 \
--tx-out $(cat payment.addr)+74999800000 \
--protocol-params-file ~/alonzo-testnet/files/pparams.json \
--out-file tx.raw
```

2. Sign the transaction.

```bash
cardano-cli transaction sign \
--testnet-magic 7 \
--signing-key-file payment.skey \
--tx-body-file tx.raw \
--out-file tx.sign
```

3. Submit the transaction.

```bash
cardano-cli transaction submit --testnet-magic 7  --tx-file tx.sign
```
- Querying the transaction:

```bash
cardano-cli query utxo --testnet-magic 7  --address $(cat payment2.addr)
```
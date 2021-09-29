## Withdrawing rewards


### Check the balance of the rewards address:
```bash
cardano-cli query stake-address-info \
--testnet-magic 8 \
--address $(cat stake.addr)
```
```json
[
    {
        "address": "stake_test1uppwpjpaquh24t6z3dn2qvk3wyy342ce7ruz8l9z0w2rmact49ugg",
        "rewardAccountBalance": 888421282907,
        "delegation": "pool1m43vhgquw3n823vnceduvvrnae9eyxakzprhcp5l3rey2ye7g6q"
    }
]
```

### Query the payment address balance

You'll withdraw rewards into a payment.addr which will pay for the transaction fees.
```bash
cardano-cli query utxo --testnet-magic 8 --address $(cat payment.addr)
```
```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
97e93bc08a2a4d22ea99e869d732c7cc13efbbe73db6e6e7413b1fd06f277907     1        1000000000000 lovelace + TxOutDatumHashNone
fa323f8759b3c523e9c802d6786b8de4f9b3030157248819975dfdbff587228c     0        1000000000000 lovelace + TxOutDatumHashNone
```

### Draft the withdraw transaction to transfer the rewards to a payment.addr

**We will be using the transaction build version instead of the build-raw command because this will give you the estimated transaction fee automatically and makes building the transaction much simpler**
```bash
cardano-cli transaction build \
--alonzo-era \
--tx-in fa323f8759b3c523e9c802d6786b8de4f9b3030157248819975dfdbff587228c#0 \
--tx-out $(cat payment.addr)+999978 \
--withdrawal $(cat stake.addr)+0 \
--invalid-hereafter 0 \
--change-address $(cat payment.addr) \
--testnet-magic 8 \
--out-file withdraw_rewards.draft
```

`Estimated transaction fee: Lovelace 165809`

### Build the final transaction.

`expr 1000000000000 - 165809 + 888421282907
1888421117098`

```bash
cardano-cli transaction build \
--alonzo-era \
--tx-in fa323f8759b3c523e9c802d6786b8de4f9b3030157248819975dfdbff587228c#0 \
--tx-in 97e93bc08a2a4d22ea99e869d732c7cc13efbbe73db6e6e7413b1fd06f277907#1 \
--tx-out $(cat payment.addr)+1888421117098 \
--withdrawal $(cat stake.addr)+888421282907 \
--invalid-hereafter 12345678 \
--change-address $(cat payment.addr) \
--testnet-magic 8 \
--out-file withdraw_rewards.raw
```

### Sign and submit the transactions
```bash
cardano-cli transaction sign \
--tx-body-file withdraw_rewards.raw  \
--signing-key-file payment.skey \
--signing-key-file stake.skey \
--testnet-magic 8 \
--out-file withdraw_rewards.signed
```
```bash
cardano-cli transaction submit \
--tx-file withdraw_rewards.signed \
--testnet-magic 8
```

```bash
cardano-cli query utxo --testnet-magic 8 --address $(cat payment.addr)
```
```
                           TxHash                                 TxIx  Amount
--------------------------------------------------------------------------------------
a84a0f59b494a83bdeee7aad7a77a0a75172a85352944d453f5bc8d20d06562f     0        111578710713 lovelace + TxOutDatumHashNone
a84a0f59b494a83bdeee7aad7a77a0a75172a85352944d453f5bc8d20d06562f     1        1888421117098 lovelace + TxOutDatumHashNone
```

## 🤡 haha this is not what I was expecting...

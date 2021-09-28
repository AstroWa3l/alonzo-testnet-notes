echo export NODE_HOME=$HOME/alonzo-testnet >> $HOME/.bashrc
echo export NODE_FILES=$HOME/alonzo-testnet/files >> $HOME/.bashrc
source $HOME/.bashrc

## Run the node

cardano-node run \
--topology /home/ubuntu/alonzo-testnet/files/alonzo-purple-topology.json \
--database-path /home/ubuntu/alonzo-testnet/db \
--socket-path /home/ubuntu/alonzo-testnet/node.socket \
--port 6001 \
--config /home/ubuntu/alonzo-testnet/files/alonzo-purple-config.json \
--shelley-vrf-key /home/ubuntu/alonzo-testnet/owner/vrf.skey \
--shelley-kes-key /home/ubuntu/alonzo-testnet/owner/kes.skey \
--shelley-operational-certificate /home/ubuntu/alonzo-testnet/owner/node.cert

## get protocol parameters
cardano-cli query protocol-parameters --testnet-magic 8 --out-file protocol.json

### Create stake address

```bash
# generate stake address
cardano-cli stake-address key-gen --verification-key-file stake.vkey --signing-key-file stake.skey
# extract readable address
cardano-cli stake-address build --stake-verification-key-file stake.vkey --out-file stake.addr --testnet-magic 8
```

### Create normal address

```bash
# generate address keys
cardano-cli address key-gen --verification-key-file payment.vkey --signing-key-file payment.skey
# extract readable address
cardano-cli address build --payment-verification-key-file payment.vkey --out-file payment.addr --testnet-magic 8
# check balance of address
cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 8
```

### Connect normal address with stake address

```bash
# create combined address
cardano-cli address build --payment-verification-key-file payment.vkey --stake-verification-key-file stake.vkey --out-file paymentwithstake.addr --testnet-magic 8
# check balance of address
cardano-cli query utxo --address $(cat paymentwithstake.addr) --testnet-magic 8
```

### get funds

```bash
# request funds
curl -k -v -XPOST "https://faucet.alonzo-purple.dev.cardano.org/send-money/$(cat paymentwithstake.addr)?apiKey=<SEEDISCORD>"
# check funds are received (may take several minutes)
cardano-cli query utxo --address $(cat paymentwithstake.addr) --testnet-magic 8
```



## Create a stake pool
### Create stake certificate
cardano-cli stake-address registration-certificate --stake-verification-key-file stake.vkey --out-file stake.cert

cardano-cli query utxo --address $(cat paymentwithstake.addr) --testnet-magic 8
> 1000000000000

# Create transaction draft
cardano-cli transaction build-raw --tx-in e70039116bd116bebf0bb8233341321caa9eebb81a2815f1a5234cf0e8d70564#0 --tx-out $(cat paymentwithstake.addr)+1000000000000 --fee 0 --out-file tx_stake.raw --certificate-file stake.cert

# Calculate fee
cardano-cli transaction calculate-min-fee --tx-body-file tx_stake.raw --tx-in-count 1 --tx-out-count 1 --witness-count 1 --testnet-magic 8 --protocol-params-file protocol.json
> 172849 Lovelace

# Calculate change (2 ada must be deposited for staking cert)
expr 1000000000000 - 172849 - 2000000
> 999997827151

# Final transaction
cardano-cli transaction build-raw --tx-in e70039116bd116bebf0bb8233341321caa9eebb81a2815f1a5234cf0e8d70564#0 --tx-out $(cat paymentwithstake.addr)+999997827151 --fee 172849 --out-file tx_stake.raw --certificate-file stake.cert

# Sign
cardano-cli transaction sign --tx-body-file tx_stake.raw --signing-key-file stake.skey --signing-key-file payment.skey --testnet-magic 8 --out-file tx_stake.signed

# Submit
cardano-cli transaction submit --tx-file tx_stake.signed --testnet-magic 8

### Find current KES Period

Lookup `slotsPerKESPeriod` and `maxKESEvolutions` at https://hydra.iohk.io/build/7189190/download/1/alonzo-purple-shelley-genesis.json

Should be 
```
"slotsPerKESPeriod": 129600,
"maxKESEvolutions": 62,
```

Find current slot with
```bash
cardano-cli query tip --testnet-magic 8
```

Calculate current KES Period
```
expr 704693 / 129600
```

Sould result in 5

### Create pool certificate
```bash
cardano-cli node key-gen --cold-verification-key-file cold.vkey --cold-signing-key-file cold.skey --operational-certificate-issue-counter-file cold.counter
cardano-cli node key-gen-VRF --verification-key-file vrf.vkey --signing-key-file vrf.skey
cardano-cli node key-gen-KES --verification-key-file kes.vkey --signing-key-file kes.skey
cardano-cli node issue-op-cert --kes-verification-key-file kes.vkey --cold-signing-key-file cold.skey --operational-certificate-issue-counter cold.counter --kes-period 5 --out-file node.cert
```

### Create pool registration
```bash

# get metadata hash
cardano-cli stake-pool metadata-hash --pool-metadata-file pool-metadata-piada.json
> 082b3ae70e47ad9954252c7986343cf39a382f8578ce639c2ed5ab4be44c42f8

# create pool-registration cert
cardano-cli stake-pool registration-certificate \
--cold-verification-key-file cold.vkey \
--vrf-verification-key-file vrf.vkey \
--pool-pledge 900000000000 \
--pool-cost 340000000 \
--pool-margin 0 \
--pool-reward-account-verification-key-file stake.vkey \
--pool-owner-stake-verification-key-file stake.vkey \
--testnet-magic 8 \
--single-host-pool-relay adasrn24x7.ddns.net \
--pool-relay-port 6001 \
--metadata-url https://piada.io/pool-metadata-piada.json \
--metadata-hash 082b3ae70e47ad9954252c7986343cf39a382f8578ce639c2ed5ab4be44c42f8 \
--out-file pool-registration.cert

curl -v -XPOST "https://faucet.alonzo-purple.dev.cardano.org/send-money/$ADDRESS?apiKey=$KEY"

jv3NBtZeaL0lZUxgqq8slTttX3BzViI7

# create delegation cert for pledge
cardano-cli stake-address delegation-certificate --stake-verification-key-file stake.vkey --cold-verification-key-file cold.vkey --out-file delegation.cert

# Submit certs
cardano-cli query utxo --address $(cat paymentwithstake.addr) --testnet-magic 8
> 999997827151

cardano-cli transaction build-raw --tx-in e70039116bd116bebf0bb8233341321caa9eebb81a2815f1a5234cf0e8d70564#0 --tx-out $(cat paymentwithstake.addr)+1000000000000 --fee 0 --out-file tx_pool.raw --certificate-file pool-registration.cert --certificate-file delegation.cert

# Calculate fee
cardano-cli transaction calculate-min-fee --tx-body-file tx_stake.raw --tx-in-count 1 --tx-out-count 1 --witness-count 1 --testnet-magic 8 --protocol-params-file protocol.json
> 173025 Lovelace

# Calculate change (500 ada must be deposited for pool registration)
expr 999497639474 - 500000000 - 187677 
> 998997451797

# build final transaction
cardano-cli transaction build-raw --tx-in e70039116bd116bebf0bb8233341321caa9eebb81a2815f1a5234cf0e8d70564#0 --tx-out $(cat paymentwithstake.addr)+999999812015 --fee 187985 --out-file tx_pool.raw --certificate-file pool-registration.cert --certificate-file delegation.cert


# Sign
cardano-cli transaction sign --tx-body-file tx_pool.raw --signing-key-file payment.skey --signing-key-file stake.skey --signing-key-file cold.skey --testnet-magic 8 --out-file tx_pool.signed

# Submit, when submit fails just use the fee from the error meesage and create/sign TX again
cardano-cli transaction submit --tx-file tx_pool.signed --testnet-magic 8

# Check if pool is registred
cardano-cli stake-pool id --cold-verification-key-file cold.vkey --output-format hex
cardano-cli stake-pool id --cold-verification-key-file cold.vkey

# Check rewards
cardano-cli query stake-distribution --testnet-magic 8 > stake-distribution.json
cardano-cli query ledger-state --testnet-magic 8 > ledger-state.json
cardano-cli query stake-address-info --testnet-magic 8 --address $(cat stake.addr)
```
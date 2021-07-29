## Setting up the raspberry pi

1. Make sure Raspberry Pi EEPROM has been flashed to allow for USB boot. For more information on how to boot from USB please see [this guide](https://docs.armada-alliance.com/learn/intermediate-guide/pi-pool-tutorial/pi-node/download-and-write-it).
2. Install and flash Ubuntu onto the SDD storage device, and make sure to add SSH file to the `system boot or boot` partition after the installation.

You can do this by unplugging the SDD and plugging it back into your computer by locating the SDD device and its partition on your computer. Typically it will be located in the `Volumes` directory.

```bash
cd /Volumes
ls
```
Once you locate the boot partition, you can copy the SSH file to the `system boot or boot` partition via the following command:
```bash
cd boot
touch ssh
```
3. After installation plug in the SDD storage device to the Raspberry Pi along with the power adapter, and then boot the Raspberry Pi.

## Setup environment

1. Build dependencies and the IOHK libsodium library

```bash
sudo apt update
sudo apt upgrade -y
reboot
```

```bash
sudo apt install automake build-essential pkg-config libffi-dev libgmp-dev libssl-dev libtinfo-dev libsystemd-dev zlib1g-dev make g++ tmux git jq wget libncursesw5 libtool autoconf
```

```bash
git clone https://github.com/input-output-hk/libsodium

cd libsodium
```
```bash
git checkout 66f017f1

./autogen.sh
```
```bash
./configure
```
```bash
make

sudo make install
```

The following steps adds the lines to your .bashrc file and sources it:
```bash
echo "export LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH"" >> .bashrc

echo "export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH"" >> .bashrc

source .bashrc
```

2. Download and copy the cardano-node and cardano-cli binaries to the Raspberry Pi' HOME directory. You can obtain the binaries from the Armada-Alliance GitHub.

```bash
wget https://github.com/AstroWa3l/alonzo-testnet-notes/blob/main/cardano-node_alonzo-white-1-1-aarch64-ubuntu-2104.tar.gz
```

3. Unpack the tar file containing the cardano-node and cardano-cli binaries.

```bash
tar -xvf cardano-node_alonzo-white-1-1-aarch64-ubuntu-2104.tar.gz
```

4. move the cardano-node and cardano-cli binaries to the `/usr/local/bin` directory.

```bash	
sudo mv cardano-node_alonzo-white-1-1-aarch64-ubuntu-2104/* /usr/local/bin/
```

5. Make some directories

```bash
mkdir -p alonzo-testnet
cd alonzo-testnet
mkdir -p files
mkdir -p scripts
```

6. Get the Alonzo testnet configuration files from the IOHK link [here](https://github.com/input-output-hk/Alonzo-testnet).

```bash
cd files
wget https://hydra.iohk.io/build/6928147/download/1/alonzo-white-config.json & wget https://hydra.iohk.io/build/6928147/download/1/alonzo-white-byron-genesis.json & wget https://hydra.iohk.io/build/6928147/download/1/alonzo-white-shelley-genesis.json & wget https://hydra.iohk.io/build/6928147/download/1/alonzo-white-alonzo-genesis.json & wget https://hydra.iohk.io/build/6928147/download/1/alonzo-white-topology.json & wget https://hydra.iohk.io/build/6928147/download/1/alonzo-white-db-sync-config.json & wget https://hydra.iohk.io/build/6928147/download/1/rest-config.json
```

7. Change back to the scripts directory and create a new file called `cardano-node.sh`

```bash
cd 
sudo nano alonzo-tesntet/scripts/cardano-node.sh
```
Copy and paste the startup script

```bash	
#!/bin/bash
cardano-node run \
 --topology /home/ubuntu/alonzo-testnet/files/alonzo-white-topology.json \
 --database-path /home/ubuntu/alonzo-testnet/db \
 --socket-path /home/ubuntu/alonzo-testnet/node.socket \
 --port 6001 \
 --config /home/ubuntu/alonzo-testnet/files/alonzo-white-config.json
 ```
 ```bash
 sudo chmod +x alonzo-testnet/scripts/cardano-node.sh
 ```

 8. Download and install gLiveView for monitoring the node.
 ```bash
 curl -s -o gLiveView.sh https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/gLiveView.sh
 ```

 ```bash
 curl -s -o env https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/env
chmod 755 gLiveView.sh
 ```
 

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

1. Install the IOHK libsodium library

2. Download and copy the cardano-node and cardano-cli binaries to the Raspberry Pi. You can obtain the binaries from the Armada-Alliance discord server.

3. Unpack the tar file containing the cardano-node and cardano-cli binaries.
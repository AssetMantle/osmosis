# Joining Mainnet

## Install Osmosis Binary

Make sure you have [installed the Osmosis Binary (CLI).](../cli/install)

## Initialize Osmosis Node

Use osmosisd to initialize your node (replace the ```NODE_NAME``` with a name of your choosing):

```bash
osmosisd init NODE_NAME
```

Download and place the genesis file in the osmosis config folder:

```
wget -O ~/.osmosisd/config/genesis.json https://github.com/osmosis-labs/networks/raw/main/osmosis-1/genesis.json
```

## Set Up Cosmovisor

Set up cosmovisor to ensure any future upgrades happen flawlessly. To install Cosmovisor:

```bash
cd $HOME
git clone https://github.com/cosmos/cosmos-sdk
cd cosmos-sdk
git checkout v0.44.0
make cosmovisor
cp cosmovisor/cosmovisor $GOPATH/bin/cosmovisor
cd $HOME
```

Create the required directories:

```bash
mkdir -p ~/.osmosisd/cosmovisor
mkdir -p ~/.osmosisd/cosmovisor/genesis
mkdir -p ~/.osmosisd/cosmovisor/genesis/bin
mkdir -p ~/.osmosisd/cosmovisor/upgrades
```

Set the environment variables:

```bash
echo "# Setup Cosmovisor" >> ~/.profile
echo "export DAEMON_NAME=osmosisd" >> ~/.profile
echo "export DAEMON_HOME=$HOME/.osmosisd" >> ~/.profile
echo "export DAEMON_ALLOW_DOWNLOAD_BINARIES=false" >> ~/.profile
echo "export DAEMON_LOG_BUFFER_SIZE=512" >> ~/.profile
echo "export DAEMON_RESTART_AFTER_UPGRADE=true" >> ~/.profile
echo "export UNSAFE_SKIP_BACKUP=true" >> ~/.profile
source ~/.profile
```
You may leave out `UNSAFE_SKIP_BACKUP=true`, however the backup takes a decent amount of time and public snapshots of old states are available.

Copy the current osmosisd binary into the cosmovisor/genesis folder:

```bash
cp $GOPATH/bin/osmosisd ~/.osmosisd/cosmovisor/genesis/bin
```

To check your work, ensure the version of cosmovisor and osmosisd are the same:

```bash
cosmovisor version
osmosisd version
```

These two command should both output 6.2.0

## Download Chain Data

Download the latest chain data from a snapshot provider. In the following commands, I will use <a href="https://quicksync.io/networks/osmosis.html" target="_blank">https://quicksync.io/networks/osmosis.html</a> to download the chain data. You may choose the default, pruned, or archive based on your needs.

Download liblz4-tool to handle the compressed file:

```bash
sudo apt-get install wget liblz4-tool aria2 -y
```

Download the chain data:

- Select the tab to the desired node type (Default, Pruned, or Archive)
- Select the tab to the region closest to you (Netherlands, Singapore, or San Francisco) and copy the commands


<!-- #region -->
::::::: tabs :options="{ useUrlFragment: false }"

:::::: tab Default
::::: tabs :options="{ useUrlFragment: false }"

:::: tab Netherlands
``` bash
URL=`curl https://quicksync.io/osmosis.json|jq -r '.[] |select(.file=="osmosis-1-default")|select (.mirror=="Netherlands")|.url'`
cd $HOME/.osmosisd/
wget -O - $URL | lz4 -d | tar -xvf -
```
::::

:::: tab Singapore
``` bash
URL=`curl https://quicksync.io/osmosis.json|jq -r '.[] |select(.file=="osmosis-1-default")|select (.mirror=="Singapore")|.url'`
cd $HOME/.osmosisd/
wget -O - $URL | lz4 -d | tar -xvf -
```
::::

:::: tab SanFrancisco
``` bash
URL=`curl https://quicksync.io/osmosis.json|jq -r '.[] |select(.file=="osmosis-1-default")|select (.mirror=="SanFrancisco")|.url'`
cd $HOME/.osmosisd/
wget -O - $URL | lz4 -d | tar -xvf -
```
::::

:::::
::::::

:::::: tab Pruned
::::: tabs :options="{ useUrlFragment: false }"

:::: tab Netherlands
``` bash
URL=`curl https://quicksync.io/osmosis.json|jq -r '.[] |select(.file=="osmosis-1-pruned")|select (.mirror=="Netherlands")|.url'`
cd $HOME/.osmosisd/
wget -O - $URL | lz4 -d | tar -xvf -
```
::::

:::: tab Singapore
``` bash
URL=`curl https://quicksync.io/osmosis.json|jq -r '.[] |select(.file=="osmosis-1-pruned")|select (.mirror=="Singapore")|.url'`
cd $HOME/.osmosisd/
wget -O - $URL | lz4 -d | tar -xvf -
```
::::

:::: tab SanFrancisco
``` bash
URL=`curl https://quicksync.io/osmosis.json|jq -r '.[] |select(.file=="osmosis-1-pruned")|select (.mirror=="SanFrancisco")|.url'`
cd $HOME/.osmosisd/
wget -O - $URL | lz4 -d | tar -xvf -
```
::::

:::::
::::::

:::::: tab Archive
::::: tabs :options="{ useUrlFragment: false }"

:::: tab Netherlands
``` bash
URL=`curl https://quicksync.io/osmosis.json|jq -r '.[] |select(.file=="osmosis-1-archive")|select (.mirror=="Netherlands")|.url'`
cd $HOME/.osmosisd/
wget -O - $URL | lz4 -d | tar -xvf -
```
::::

:::::
::::::

:::::::

<!-- #endregion -->

## Set Up Osmosis Service

Set up a service to allow cosmovisor to run in the background as well as restart automatically if it runs into any problems:

```bash
echo "[Unit]
Description=Cosmovisor daemon
After=network-online.target
[Service]
Environment="DAEMON_NAME=osmosisd"
Environment="DAEMON_HOME=${HOME}/.osmosisd"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_LOG_BUFFER_SIZE=512"
Environment="UNSAFE_SKIP_BACKUP=true"
User=$USER
ExecStart=${HOME}/go/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=infinity
LimitNPROC=infinity
[Install]
WantedBy=multi-user.target
" >cosmovisor.service
```

Move this new file to the systemd directory:

```bash
sudo mv cosmovisor.service /lib/systemd/system/cosmovisor.service
```

## Start Osmosis Service

Reload and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl restart systemd-journald
sudo systemctl start cosmovisor
```

Check the status of the service:

```bash
sudo systemctl status cosmovisor
```

To see live logs of the service:

```bash
journalctl -u cosmovisor -f
```
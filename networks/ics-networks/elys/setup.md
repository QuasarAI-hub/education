### System requirements

- OS: Ubuntu server 22 LTS
- CPU: 4 cores
- RAM: 4 Gb
- Storage: 500 Gb Nvme

### Update system and install build tools

Ensure your system is up to date and has all the necessary tools for the installation:

```bash
sudo apt update && sudo apt upgrade -y && sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc chrony liblz4-tool -y
```

### Install Go

Replace `VERSION` with the desired Go version

```bash
VERSION="1.23.0"
cd $HOME
wget "https://golang.org/dl/go$VERSION.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VERSION.linux-amd64.tar.gz"
rm "go$VERSION.linux-amd64.tar.gz"
echo "export PATH=\$PATH:/usr/local/go/bin:\$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
```

### Install node

```bash
cd $HOME
mkdir -p $HOME/src
cd src
git clone https://github.com/elys-network/elys
cd elys
git checkout v1.4.0
make install
```

### Initialize Node

Replace <node_name>

```bash
~/go/bin/elysd init <node_name> --chain-id="elys-1"
```

### Download genesis.json

```bash
cd $HOME/.elys/config/
rm -f genesis.json
wget https://github.com/elys-network/networks/blob/main/mainnet/genesis.json
```

### Create Service

#### Cosmovisor:

If you don’t have cosmovisor

```bash
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
```

You can find cosmovisor binary in `~/go/bin/` folder. After that you should create

```bash
mkdir -p $HOME/.elys/cosmovisor/genesis/bin && mkdir -p $HOME/.elys/cosmovisor/upgrades
cp $HOME/go/bin/elysd $HOME/.elys/cosmovisor/genesis/bin/elysd
```

Set up service:

```bash
SERVICE_FILE="/etc/systemd/system/elysd.service"
DAEMON_HOME="$HOME/.elys"

sudo bash -c "cat <<EOL > $SERVICE_FILE
[Unit]
Description=elysd Daemon cosmovisor
After=network-online.target

[Service]
User=$USER
ExecStart=/home/$USER/go/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=4096
Environment=\"DAEMON_NAME=elysd\"
Environment=\"DAEMON_HOME=$DAEMON_HOME\"
Environment=\"DAEMON_ALLOW_DOWNLOAD_BINARIES=false\"
Environment=\"DAEMON_RESTART_AFTER_UPGRADE=true\"
Environment=\"DAEMON_LOG_BUFFER_SIZE=512\"

[Install]
WantedBy=multi-user.target
EOL"

sudo chmod 644 $SERVICE_FILE
sudo systemctl daemon-reload
```

#### Simple service file:

Set up service:

```bash
SERVICE_FILE="/etc/systemd/system/elysd.service"
USER=$(whoami)

sudo cat <<EOL > $SERVICE_FILE
[Unit]
Description=elysd Daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$HOME/go/bin/elysd start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOL

sudo chmod 644 $SERVICE_FILE
sudo systemctl daemon-reload
```

### Add peers

```bash
PEERS=f174206d6b3dbc2dd17cdd884bdfc6ad37268a09@67.218.8.88:26665,5545dc6fa6537ce464a49593bac02258fd963e57@67.218.8.88:26665,cd2311ffdae014daff80c343c26a393e714c7973@172.31.23.120:26656
sed -i.bak -e "s/^persistent*peers *=.\_/persistent_peers = \"$PEERS\"/" $HOME/.elys/config/config.toml
```

### Sync node:

```bash
SNAP_RPC="https://dungeon.rpc.quasarstaking.ai:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+)._$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+)._$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.elys/config/config.toml
```

Check node status

```bash
curl -s http://localhost:26657/status | jq '.result.sync_info.catching_up'
```

### Start service

```bash
sudo systemctl enable elysd.service && sudo systemctl start elysd.service && journalctl -u elysd.service -f
```

## If need upgrade

```bash
cd $HOME/src/elys
git fetch origin && git checkout v3.0.0
make build
cd build
```

Depends of type to running a node (simple running or with cosmovisor)

Cosmovisor:

```bash
mkdir -p $HOME/.elys/cosmovisor/upgrades/v3.0.0/bin/
mv elysd $HOME/.elys/cosmovisor/upgrades/v3.0.0/bin/
```

Simple running:

```bash
rm -f $HOME/go/bin/elysd
mv elysd $HOME/go/bin/
```

## Creating validator

Replace <wallet>

```bash
PUBKEY=$(~/go/bin/elysd keys show <wallet> --pubkey)
```

Registering validator in cosmoshub-4

```bash
gaiad tx provider opt-in 21 $PUBKEY --from <wallet> --fees=10000uatom --gas=auto --gas-adjustment 1.5 --chain-id cosmoshub-4
```

Creating a validator on the elys-1 network

```bash
AMOUNT="1000000uelys"
MONIKER=<validator-moniker>
IDENTITY=<identity>
WEBSITE=<validator-website>
DETAILS=<validator-details>
COMMISSION_RATE="0.05"
COMMISSION_MAX_RATE="0.2"
COMMISSION_MAX_CHANGE_RATE="0.05"
MIN_SELF_DELEGATION="1"

cat <<EOL > validator.json
{
    "pubkey": "$PUBKEY",
    "amount":  "$AMOUNT",
    "moniker": "$MONIKER",
    "website": "$WEBSITE",
    "details": "$DETAILS",
    "identity": "$IDENTITY",
    "commission-rate": "$COMMISSION_RATE",
    "commission-max-rate": "$COMMISSION_MAX_RATE",
    "commission-max-change-rate": "$COMMISSION_MAX_CHANGE_RATE",
    "min-self-delegation": "$MIN_SELF_DELEGATION"
}
EOL

elysd tx staking create-validator validator.json --from <wallet> --chain-id "elys-1"
```

# Useful commands

## Key mangment

### Add new key

```bash
elysd keys add wallet
```

### Recover existing key

```bash
elysd keys add wallet --recover
```

### List all keys

```bash
elysd keys list
```

### Delete key

```bash
elysd keys delete wallet
```

### Export key to a file

```bash
elysd keys export wallet
```

### Import key from a file

```bash
elysd keys import wallet wallet.backup
```

## Validator mangment

### Edit validator

```bash
elysd tx staking edit-validator \
--new-moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id cosmoshub-4 \
--commission-rate 0.05 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.005uatom
```

### Unjail validator

```bash
elysd tx slashing unjail --from wallet --chain-id cosmoshub-4 --gas-adjustment 1.4 --gas auto --gas-prices 0.05uatom
```

### Jail reason

```bash
elysd query slashing signing-info $(elysd comet show-validator)
```

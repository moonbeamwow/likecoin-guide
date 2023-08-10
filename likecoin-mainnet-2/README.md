### This guide contains step-by-step instructions on how to install and synchronize your own node for the likecoin-mainnet-2 chain
#### Node installation
##### Update necessary packages
```bash
sudo apt update
sudo apt upgrade --yes
```
##### Install git, curl, jq and make
```bash
sudo apt install jq git curl make --yes
```
##### Install Go
```bash
sudo snap install go --classic && \
echo 'export GOPATH="$HOME/go"' >> ~/.profile && \
echo 'export GOBIN="$GOPATH/bin"' >> ~/.profile && \
echo 'export PATH="$GOBIN:$PATH"' >> ~/.profile && \
source ~/.profile && \
go version
```
##### Clone LikeCoin repository and install binary
```bash
git clone https://github.com/likecoin/likecoin-chain
cd likecoin-chain
git checkout v$(curl -s http://207.180.238.137:26657/abci_info | jq -r .result[].version)
make install
```
##### Download genesis and init your node
```bash
liked init <moniker> --chain-id likecoin-mainnet-2
liked keys add <keyname>
wget https://raw.githubusercontent.com/likecoin/mainnet/master/genesis.json -O /root/.liked/config/genesis.json
```
##### Setup liked service
```bash
sudo tee /etc/systemd/system/liked.service > /dev/null <<EOF
[Unit]
Description=LikeCoin Node Daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$(which liked) start
Restart=on-failure
RestartSec=3
LimitNOFILE=100000

[Install]
WantedBy=multi-user.target
EOF
```
##### Reload and enable your daemon
```bash
sudo systemctl daemon-reload
sudo systemctl enable likied
```
#### Node synchronization
##### For your convenience, the moonbeam validator provides the state sync service, which allows you to quickly synchronize your node with the chain
##### Stop your node and do unsafe-reset
```bash
systemctl stop liked
liked tendermint unsafe-reset-all --home $HOME/.liked
```
##### Set up variables
```bash
SNAP_RPC=http://207.180.238.137:26657
peers="d85e025a1b815412a76c0fe5fa5c9d6ea0b5dfae@207.180.238.137:26656"
sed -i -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.liked/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
##### Add variables to config.toml
```bash
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; 
\s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; 
\s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; 
\s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.liked/config/config.toml
```
##### Restart your node and watch logs
```
sudo systemctl restart liked
journalctl -u liked -f -o cat
```
##### You can check node sync status with command
```
curl -s localhost:26657/status
```
##### Validator creation
###### When your node is sysnced, you are allowed to create your own validator:
```liked tx staking create-validator \
--amount=<amount>nanolike \
--pubkey=$(liked tendermint show-validator) \
--moniker=$MONIKER \
--commission-rate="0.10" \
--commission-max-rate="0.50" \
--commission-max-change-rate="0.05" \
--min-self-delegation="<amount>" \
--chain-id="likecoin-mainnet-2" \
--from=<key_name> \
--keyring-backend=file \
--gas-prices 1nanolike
```
Track your node on this page: https://likecoin.bigdipper.live/validators.

### RPC/API/State sync services:
RPC endpoint: http://207.180.238.137:26657
API endpiont: http://207.180.238.137:1317
gRPC/web endpoints: http://207.180.238.137:9090 / http://207.180.238.137:9091

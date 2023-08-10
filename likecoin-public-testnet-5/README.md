### This guide contains step-by-step instructions on how to install your own node for the likecoin-public-testnet-5 chain
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
##### Clone LikeCoin repository
```bash
git clone https://github.com/likecoin/likecoin-chain.git --branch release/v3.x --single-branch
```
##### Download genesis and init your node
```bash
export MONIKER='<validator name>'
export GENESIS_URL='https://raw.githubusercontent.com/likecoin/testnets/master/likecoin-public-testnet-5/genesis.json'
export LIKED_SEED_NODES='1ca24cb5e744287284372cfdfe545f9ee16703c6@20.6.73.206:26656,49976c3bd43da9271f226cbedf02d4b6b8fc880c@35.233.143.230:26656'
export LIKED_VERSION='4.0.0'
```
##### Execute setup script
```bash
cd ~/likecoin-chain
make -C deploy setup-node
```
##### Activate liked service and run it
```bash
cd ~/likecoin-chain
make -C deploy initialize-systemctl
make -C deploy start-node
```

#### Useful commands
##### Stop liked service
```bash
systemctl stop liked
```
##### Restart liked service
```bash
systemctl restart liked
```
##### Reset all node data
```bash
liked tendermint unsafe-reset-all --home $HOME/.liked
```
##### Watch logs
```bash
journalctl -u liked -f -o cat
```
##### See sync status
```bash
curl -s localhost:26657/status
```
##### Create validator
```bash
liked tx staking create-validator \
--amount=<amount>nanoekil \
--pubkey=$(liked tendermint show-validator) \
--moniker=<moniker> \
--commission-rate="0.10" \
--commission-max-rate="0.50" \
--commission-max-change-rate="0.05" \
--min-self-delegation="<amount>" \
--chain-id="likecoin-public-testnet-5" \
--from=<key_name> \
--keyring-backend=file \
--gas-prices 1nanoekil
```
Track your node on this page: [https://likecoin.bigdipper.live/validators](https://testnet.bigdipper.live/likecoin)https://testnet.bigdipper.live/likecoin.

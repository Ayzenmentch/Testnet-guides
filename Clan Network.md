## Install dependencies:
```
sudo apt update && sudo apt upgrade -y
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils -y
```
## Install Go:
```
wget -O go1.18.linux-amd64.tar.gz https://go.dev/dl/go1.18.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.linux-amd64.tar.gz && rm go1.18.linux-amd64.tar.gz
cat <<'EOF' >> $HOME/.bash_profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
. $HOME/.bash_profile
cp /usr/local/go/bin/go /usr/bin
go version
```
## Clone git repository:
```
git clone https://github.com/ClanNetwork/clan-network
cd clan-network
git fetch origin --tags
git checkout v1.0.4-alpha
```
## Install:
```
make install
cp $HOME/go/bin/cland /usr/local/bin
```
## Verify installation:
```
#Verify that everything is OK.

cland version --long
name: Clan-Network
server_name: clan-networkd
version: 1.0.4-alpha
commit: 7a6a92d782c978ac730e337b28d2bc927e809739
build_tags: ""
go: go version go1.18 darwin/amd64
```
## Add variables:
```
echo 'export CLAN_NODENAME="Your Node name"'>> $HOME/.bash_profile
echo 'export ClAN_WALLET="Your Clan Wallet name"'>> $HOME/.bash_profile
echo 'export CLAN_CHAIN="playstation-2"' >> $HOME/.bash_profile
. $HOME/.bash_profile

# check variables:
echo $CLAN_NODENAME
echo $ClAN_WALLET
echo $CLAN_CHAIN
```
## Init:
```
cland init $CLAN_NODENAME --chain-id $CLAN_CHAIN
```
## Generate keys:
```
cland keys add $ClAN_WALLET
```

## Download genesis:
```
curl https://raw.githubusercontent.com/ClanNetwork/testnets/main/$CHAIN_ID/genesis.json > ~/.clan/config/genesis.json
```
## Unsafe restart all:
```
cland tendermint unsafe-reset-all --home ~/.clan
```
## Configure your node:
```
CHAIN_REPO="https://raw.githubusercontent.com/ClanNetwork/testnets/main/$CHAIN_ID" && \
export PEERS="$(curl -s "$CHAIN_REPO/persistent-peers.txt")"
# check it worked
echo $PEERS

sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ~/.clan/config/config.toml
```
## Set 0 gas prices:
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uclan\"/" ~/.clan/config/app.toml
```
## install service to run the node:
```
sudo tee /etc/systemd/system/cland.service > /dev/null <<EOF
[Unit]
Description=Clan Daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cland) start
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable cland
sudo systemctl restart cland
```
## Check your node logs:
```
journalctl -u cland -f
```
## Status of sinchronization:
```
cland status 2>&1 | jq .SyncInfo

curl http://localhost:26657/status | jq .result.sync_info.catching_up
```
### Faucet: https://faucet-testnet.clan.network/

## Сheck your balance:
```
cland q bank balances $(cland keys show $ClAN_WALLET -a)
```
## Create validator:
```
cland tx staking create-validator \
  --amount 1000000000uclan \
  --commission-max-change-rate=0.1 \
  --commission-max-rate=0.20 \
  --commission-rate=0.1 \
  --min-self-delegation=1 \
  --details "" \
  --website=""\
  --identity= ""\
  --pubkey=$(cland tendermint show-validator) \
  --moniker=$CLAN_NODENAME \
  --chain-id=$CLAN_CHAIN \
  --gas-prices=0uclan \
  --from=$ClAN_WALLET
```

## Check your node status:
```
curl localhost:26657/status
```
## Collect rewards:
```
cland tx distribution withdraw-all-rewards \
 --chain-id=$CLAN_CHAIN \
 --from $ClAN_WALLET \
 --gas auto \
 --gas-prices=0uclan
```
## Delegate tokens to your validator:
```
cland tx staking delegate $(cland keys show $ClAN_WALLET --bech val -a) <amountuclan> \
--chain-id=$CLAN_CHAIN \
--from=$ClAN_WALLET \
--gas auto \
--gas-prices=0uclan
```
## Unjail:
```
cland tx slashing unjail \
--chain-id $CLAN_CHAIN \ 
--from $ClAN_WALLET\ 
--gas=auto \ 
--gas-prices=0uclan
```
## Stop the node:
```
sudo systemctl stop cland
```
## Delete node files and directories:
```
sudo systemctl stop cland
sudo systemctl disable cland
rm /etc/systemd/system/cland.service
rm -Rvf $HOME/clan
rm -Rvf $HOME/.clan
```

Official website: https://www.clan.network/

Official documentations:
1. https://docs.clan.network/nodes-and-validators/joining-testnets#cland-installation
2. https://secretnodes.com/clan/chains/playstation-2/validators?node_filter=Active

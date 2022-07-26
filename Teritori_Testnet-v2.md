# Teritori_Testnet-v2
## Update the packeges
```
sudo apt update && sudo apt upgrade -y
```
## Install dependencies:
```
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils -y
-//- apt install build-essential git curl gcc make jq -y
```
## Install Go:
```
wget -O go1.18.2.linux-amd64.tar.gz https://golang.org/dl/go1.18.2.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.2.linux-amd64.tar.gz && rm go1.18.2.linux-amd64.tar.gz

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
## Clone the Teritori git repository and install the v2 of testnet :
```
git clone https://github.com/TERITORI/teritori-chain; \
cd teritori-chain; \
git checkout teritori-testnet-v2; \
make install
cp $HOME/go/bin/teritorid /usr/local/bin
```
## Verify the installation:
```
teritorid version
```
## Should return  teritori-testnet-v2-0f4e5cb1d529fa18971664891a9e8e4c114456c6

## Add variables:
```
echo 'export TER_NODENAME="Your Node name"'>> $HOME/.bash_profile
echo 'export TER_WALLET="Your Teritori Wallet name"'>> $HOME/.bash_profile
echo 'export TER_CHAIN="teritori-testnet-v2"' >> $HOME/.bash_profile
. $HOME/.bash_profile
```
## check variables:
```
echo $TER_MONIKER
echo $TER_WALLET
echo $TER_CHAIN
```
## Init the chain:
```
teritorid init $TER_MONIKER --chain-id teritori-testnet-v2
```
## Generate keys:
```
teritorid key add $TER_WALLET
```
## Download genesis and addrbook
```
wget -qO $HOME/.teritorid/config/genesis.json "https://raw.githubusercontent.com/TERITORI/teritori-chain/main/testnet/teritori-testnet-v2/genesis.json"
wget -qO $HOME/.teritorid/config/addrbook.json "https://raw.githubusercontent.com/StakeTake/guidecosmos/main/teritori/teritori-testnet-v2/addrbook.json"
```
# Add peers in the config file:
```
sed -i.bak 's/persistent_peers =.*/persistent_peers = "0b42fd287d3bb0a20230e30d54b4b8facc412c53@176.9.149.15:26656,2371b28f366a61637ac76c2577264f79f0965447@176.9.19.162:26656,2f394edda96be07bf92b0b503d8be13d1b9cc39f@5.9.40.222:26656"/' $HOME/.teritorid/config/config.toml
```
# Set minimum gas prices:
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0ustrd\"/" $HOME/.stride/config/app.toml
```
## Configure pruning:
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.teritorid/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.teritorid/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.teritorid/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.teritorid/config/app.toml
```
## Create service file
```
tee <<EOF >/dev/null /etc/systemd/system/teritorid.service
[Unit]
Description=Teritori Cosmos daemon
After=network-online.target

[Service]
User=$USER
ExecStart=/home/$USER/go/bin/teritorid start
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF

systemctl enable teritorid
systemctl daemon-reload
systemctl restart teritorid
```
## check the logs:
```
journalctl -u strided -f --output cat
```
## Check the sync status
```
strided status 2>&1 | jq .SyncInfo
```

## Join the Teritori Discord and request fund on the Faucet channel using this command:
```
$request <YOUR_TERITORI_ADDRESS>
```
## You can check if you have received fund once your node will be synched using this CLI command:
```
teritorid query bank balances <YOUR_TERITORI_ADDRESS> --chain-id teritori-testnet-v2
```
## Once the fund are received and chain is synchronized you can create your validator:
```
teritorid tx staking create-validator \
 --commission-max-change-rate=0.01 \
 --commission-max-rate=0.2 \
 --commission-rate=0.05 \
 --amount 1000000utori \
 --pubkey=$(teritorid tendermint show-validator) \
 --moniker=<YOUR_MONIKER> \
 --chain-id=teritori-testnet-v2 \
 --details="<DESCRIPTION_OF_YOUR_VALIDATOR>" \
 --security-contact="<YOUR_EMAIL_ADDRESS" \
 --website="<YOUR_WEBSITE>" \
 --identity="<YOUR_KEYBASE_ID>" \
 --min-self-delegation=1000000 \
 --from=<YOUR_KEY>
```
## Check your node status:
```
curl localhost:26657/status
```
## Add liquid stake:
```
strided tx stakeibc liquid-stake 1000 uatom --from $STRIDE_WALLET --chain-id $STRIDE_CHEIN
```
## Collect rewards:
```
 teritorid tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$TER_CHAIN --gas=auto
```
## Delegate tokens to your validator:
```
teritorid tx staking delegate $TER_VALOPER_ADDRESS 10000000utori --from=$WALLET --chain-id=$TER_CHAIN --gas=auto
```
## Unjail:
```
teritorid tx slashing unjail \
  --broadcast-mode=block \
  --from=$TER_WALLET \
  --chain-id=$TER_CHAIN \
  --gas=auto
```
##  Usefull commands
## Node info
```
teritorid status 2>&1 | jq .NodeInfo
```
## Synchronization status
```
teritorid status 2>&1 | jq .SyncInfo
curl http://localhost:26657/status | jq .result.sync_info.catching_up
```
## Validator info
```
teritorid status 2>&1 | jq .ValidatorInfo
```
## Show node id
```
teritorid tendermint show-node-id
```
## Check logs
```
journalctl -u strided -f --output cat
```
## Stop service
```
sudo systemctl stop teritorid
```
## Restart service
```
sudo systemctl restart teritorid
```

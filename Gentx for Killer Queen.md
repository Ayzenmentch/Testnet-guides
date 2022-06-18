
## Killer Queen 
``
## Install dependencies:
```
sudo apt update && sudo apt upgrade -y
sudo apt-get install jq curl git gcc g++ make -y
```
## Install Go:
```
get https://golang.org/dl/go1.18.1.linux-amd64.tar.gz; \
rm -rv /usr/local/go; \
tar -C /usr/local -xzf go1.18.1.linux-amd64.tar.gz && \
rm -v go1.18.1.linux-amd64.tar.gz && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```
## Clone git repository:
```
cd $HOME
git clone <https://github.com/ingenuity-build/quicksilver.git> --branch v0.3.0
cd quicksilver
make build
chmod +x ./build/quicksilverd && mv ./build/quicksilverd /usr/local/bin/quicksilverd
```
## Add variables:
``
echo 'export YOUR_MONIKER="Your Node name"'>> $HOME/.bash_profile
echo 'export YOUR_WALLET="Your Clan Wallet name"'>> $HOME/.bash_profile
echo 'export CHAIN_ID="killerqueen-1"' >> $HOME/.bash_profile
. $HOME/.bash_profile
``
## Init:
```
quicksilverd init $YOUR_MONIKER --chain-id $CHAIN_ID
```

## Generate keys:
```
quicksilverd keys add $YOUR_WALLET --recover
```
## Add genesis accaunt:
```
quicksilverd add-genesis-account $YOUR_WALLET 100000000uqck
```
## Generate gentx
```
quicksilverd gentx $YOUR_WALLET 100000000uqck \
--chain-id=CHAIN_ID \
--commission-rate=0.05 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.1 \
--pubkey $(quicksilverd tendermint show-validator) \
--moniker $YOUR_MONIKER

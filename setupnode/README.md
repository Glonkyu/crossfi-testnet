# CrossFi Testnet Full Node Installation and Join Validator Guide

This guide provides detailed steps on setting up a Full Node for the CrossFi Testnet. By following this guide, you will install all necessary dependencies, configure, start the node and create your validator.

## Setting Up Variables

First, define a nodename for your validator that will appear in the explorer.

```bash
NODENAME="<Your_Nodename_Moniker>"
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
```

### Save Variables to System

Store your wallet and CrossFi chain ID as environment variables.

```bash
echo "export WALLET=wallet" >> $HOME/.bash_profile
echo "export CROSSFI_CHAIN_ID=crossfi-evm-testnet-1" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Update Packages

Ensure your package lists and installed packages are updated.

```bash
sudo apt update && sudo apt upgrade -y
```

## Install Dependencies

Install necessary packages required for building and running the node.

```bash
sudo apt install curl build-essential git wget jq make gcc tmux net-tools ccze -y
```

## Install Go

Install Go, required for building the node software.

```bash
if ! [ -x "$(command -v go)" ]; then
    ver="1.20.2"
    cd $HOME
    wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
    sudo rm -rf /usr/local/go
    sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
    rm "go$ver.linux-amd64.tar.gz"
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
    source ~/.bash_profile
fi
```

## Download and Setup Binary

Fetch and set up the CrossFi node software.

```bash
cd $HOME
wget https://github.com/crossfichain/crossfi-node/releases/download/v0.3.0-prebuild3/crossfi-node_0.3.0-prebuild3_linux_amd64.tar.gz
tar -xvf crossfi-node_0.3.0-prebuild3_linux_amd64.tar.gz
chmod +x $HOME/bin/crossfid
mv $HOME/bin/crossfid $HOME/go/bin
rm -rf crossfi-node_0.3.0-prebuild3_linux_amd64.tar.gz $HOME/bin
```

## Initialize Application

Set up the initial blockchain state.

```bash
crossfid init $NODENAME --chain-id $CROSSFI_CHAIN_ID
rm -rf testnet ~/.mineplex-chain
git clone https://github.com/crossfichain/testnet.git
mv $HOME/testnet/ $HOME/.mineplex-chain/
```

## Download Configuration Files

Fetch the genesis and address book files.

```bash
wget https://service.jembutkucing.tech/crossfi-t/addrbook.json -O $HOME/.mineplex-chain/config/genesis.json
wget https://service.jembutkucing.tech/crossfi-t/addrbook.json -O $HOME/.mineplex-chain/config/addrbook.json
```

## Configure Node Settings

Adjust gas pricing and disable indexing for performance.

```bash
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "10000000000000mpx"|g' $HOME/.mineplex-chain/config/app.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.mineplex-chain/config/config.toml
```

### Config Pruning

Configure state pruning to manage state size.

```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.mineplex-chain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.mineplex-chain/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.mineplex-chain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.mineplex-chain/config/app.toml
```

## Create System Service

Set up a systemd service to keep your node running smoothly.

```bash
sudo tee /etc/systemd/system/crossfid.service > /dev/null <<EOF
[Unit]
Description=Crossfi Node
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.mineplex-chain
ExecStart=$(which crossfid) start --home $HOME/.mineplex-chain
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Register and Start Service

Enable and start your node.

```bash
sudo systemctl daemon-reload
sudo systemctl enable crossfid
sudo systemctl restart crossfid && sudo journalctl -u crossfid -f -o cat
```

### **Check Sync Status**

Verify that your node has fully synced with the network.

```bash
crossfid status 2>&1 | jq .SyncInfo
```

## **Wallet and Validator Setup**

### **Create or Restore a Wallet**

Create a new wallet or restore an existing one. **Do not forget to save the mnemonic securely.**

```bash
# Create a new wallet
crossfid keys add $WALLET

# Restore an existing wallet
crossfid keys add $WALLET --recover
```

### **Save Wallet and Validator Addresses**

Extract and store your wallet and validator addresses.

```bash
WALLET_ADDRESS=$(crossfid keys show $WALLET -a)
VALOPER_ADDRESS=$(crossfid keys show $WALLET --bech val -a)
echo "export WALLET_ADDRESS=$WALLET_ADDRESS" >> $HOME/.bash_profile
echo "export VALOPER_ADDRESS=$VALOPER_ADDRESS" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
### **Fund Your Wallet**

Before creating a validator, ensure your wallet is funded. Check your balance and request funds from the [Faucet Here](https://docs.crossfi.org/crossfi-foundation/xfi-pad#discord-faucet)
 if necessary.

```bash
crossfid query bank balances $WALLET_ADDRESS
```
### **Create Validator**

Run the following command to create your validator:

```bash
crossfid tx staking create-validator \
  --amount 1000000000000000000mpx \
  --from $WALLET \
  --commission-rate 0.1 \
  --commission-max-rate 0.2 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --pubkey $(crossfid tendermint show-validator) \
  --moniker "test" \
  --identity "" \
  --details "" \
  --chain-id $CROSSFI_CHAIN_ID \
   --gas 250000 --gas-adjustment 1.5 --gas-prices 5000000000mpx \
  -y
```

### **Create Validator**

Run the following command to create your validator:

```bash
crossfid tx staking create-validator \
  --amount 1000000mpx \
  --from $WALLET \
  --commission-rate 0.1 \
  --commission-max-rate 0.2 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --pubkey $(crossfid tendermint show-validator) \
  --moniker "test" \
  --identity "" \
  --details "" \
  --chain-id $CROSSFI_CHAIN_ID \
  --gas auto --gas-adjustment 1.5 --gas-prices 10000000000000mpx \
  -y
```

### **Monitoring**

Regularly check the logs using `journalctl` to monitor the node’s operation and catch any issues early. Keeping a close eye on your node activities and performance is crucial for maintaining a healthy validator status.

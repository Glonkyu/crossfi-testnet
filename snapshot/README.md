# This is a guide for running nodes with the snapshot method, the goal is to synchronize more quickly to reach the latest block

# First you have to stop the node depending on how you start
```
CTRL+C
```
or
```
sudo systemctl stop crossfid
```

# Backup the state file and reset data folder
```
cp $HOME/.mineplex-chain/data/priv_validator_state.json $HOME/.mineplex-chain/priv_validator_state.json.backup
rm -rf $HOME/.mineplex-chain/data
```
# Install lz4
```
apt update && apt-get install lz4 -y
```
# Download the latest snapshot, extract the file then move back the backed up state file
```
curl -o - -L https://service.jembutkucing.tech/crossfi-t/crossfi-testnet-snapshot.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.mineplex-chain --strip-components 2
mv $HOME/.mineplex-chain/priv_validator_state.json.backup $HOME/.mineplex-chain/data/priv_validator_state.json
```

# Download new addressbook
```
wget -O $HOME/.mineplex-chain/config/addrbook.json "http://service.jembutkucing.tech/crossfi-t/addrbook.json"
```

# Restart the service and check the logs
```
crossfid start
```
or
```
sudo systemctl start crosfid && sudo journalctl -u crossfid -f
```





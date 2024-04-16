**Stop Node And Reset data**
```
CTRL+C
```
or
```
sudo systemctl stop crossfid
crossfid tendermint unsafe-reset-all --home ~/.mineplex-chain/ --keep-addr-book
```

**Setup State Sync**
```
SNAP_RPC="https://crossfi-rpc.jembutkucing.tech:443"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
 
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" ~/.mineplex-chain/config/config.toml
```

**Restart Node and check the logs**
```
crossfid start
```
or
```
sudo systemctl restart crossfid && journalctl -u crossfid -f
```

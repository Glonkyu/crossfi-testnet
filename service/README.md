# **CrossFi Testnet Resource Directory**
Below is a list of resources available for interacting with the CrossFi infrastructure through various network protocols and application programming interfaces (APIs). Each resource provides access to specific functionalities tailored to meet your system integration and development needs.
| Service | URL |
| ----------- | ----------- |
| RPC | https://crossfi-rpc.jembutkucing.tech |
| API | https://crossfi-api.jembutkucing.tech |
| JsonRPC | https://crossfi-jsonrpc.jembutkucing.tech |
| gRPC | https://crossfi-grpc.jembutkucing.tech |
| Explorer | https://crossfi-explorer.jembutkucing.tech |
| Guide | https://github.com/Glonkyu/crossfi-testnet/blob/master/setupnode/README.md | 
| Snapshot | https://github.com/Glonkyu/crossfi-testnet/tree/master/snapshot |
| State Sync | https://github.com/Glonkyu/crossfi-testnet/tree/master/statesync |

**Genesis file**
```
wget -O $HOME/.mineplex-chain/config/genesis.json https://service.jembutkucing.tech/crossfi-t/genesis.json
```
**Addressbook**
```
wget -O $HOME/.mineplex-chain/config/addrbook.json  https://service.jembutkucing.tech/crossfi-t/addrbook.json
```
**Peers**
```
PEERS="$(curl -sS https://crossfi-rpc.jembutkucing.tech:443/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}' | sed -z 's|\n|,|g;s|.$||')"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $HOME/.mineplex-chain/config/config.toml
```

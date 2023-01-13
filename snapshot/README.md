
# OKP4 SNAPSHOT

## Snapshot URL
Pruned daily snapshot here: [https://okp4.westaking.io/okp4/snapshot/](https://okp4.westaking.io/okp4/snapshot/)

## Instruction
```bash
# GET SNAPSHOT FILENAME
SNAPSHOT_FILENANE=$(curl -s https://okp4.westaking.io/okp4/snapshot/  | egrep -o ">okp4.*.tar.lz4" | tr -d ">")

# STOP CHAIN
sudo systemctl stop okp4

# RESET CHAIN
okp4d tendermint unsafe-reset-all --keep-addr-book

# SET CHAIN DIRECTORY
cd ~/.okp4d

# DOWNLOAD & EXTRACT CHAIN DATA
wget -O - https://okp4.westaking.io/okp4/snapshot/$SNAPSHOT_FILENANE| lz4 -d | tar -xvf -

# RESTART OKP4 SERVICE
sudo systemctl start okp4
```

# OKP4 STATESYNC

## state-sync server setup
app.toml
```toml
# Prune Strategy
pruning = "custom"
pruning-keep-every = 500

# State-Sync Snapshot Strategy
snapshot-interval = 500
snapshot-keep-recent = 3
```

## Instruction
```bash
SNAP_RPC="https://okp4.westaking.io:443"
SNAP_INTERVAL=500

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - SNAP_INTERVAL)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT
echo $BLOCK_HEIGHT
echo $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.okp4d/config/config.toml

# ENABLE SNAPSHOT MANAGER
sed -E -i "/snapshot-interval = /c\snapshot-interval = $SNAP_INTERVAL" $HOME/.okp4d/config/app.toml
sed -E -i "/snapshot-keep-recent = /c\snapshot-keep-recent = 3" $HOME/.okp4d/config/app.toml

# STOP CHAIN
sudo systemctl stop okp4

# RESET CHAIN
okp4d tendermint unsafe-reset-all --keep-addr-book

# RESTART OKP4 SERVICE
sudo systemctl start okp4
```


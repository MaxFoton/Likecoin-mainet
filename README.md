# Likecoin-mainnet-2

![Alt-logo](https://raw.githubusercontent.com/MaxFoton/Likecoin-mainnet/main/LikeCoin.png)

`Validator` maxfoton

`Delegate` https://ping.pub/likecoin/staking/likevaloper1fhucdvrtdpjzwsld3ge5yccr7tgvdh9p0fca5j

`Valoper` likevaloper1fhucdvrtdpjzwsld3ge5yccr7tgvdh9p0fca5j

**Links**

- `Website` - https://like.co/
- `Twitter` - https://twitter.com/likecoin
- `Github` - https://github.com/likecoin
- `Discord` - https://discord.gg/likecoin
- `Blog` - https://blog.like.co/
- `Newsletter` - https://likecoin.substack.com/

**Links mainnet**
https://docs.like.co/validator/likecoin-chain-node/setup-a-node

**RPC**
`RPC 95.165.89.222:26557`

**Peer**

- `Peer` ccb2af8f2d52324a25ef42a3d2b0b9852675d55b@46.138.245.164:26556

Genesis and addrbook

- `Genesis` https://raw.githubusercontent.com/likecoin/mainnet/master/genesis.json
- `Addrbook` https://raw.githubusercontent.com/MaxFoton/Likecoin-mainnet/main/addrbook.json

**Explorer**

- `Mintscan` https://www.mintscan.io/likecoin
- `Ping` https://ping.pub/likecoin

## Installation Steps

- Ensure that you have the following installed on your machine:

Go (version 1.19+) https://golang.org/doc/install
```
sudo apt update && sudo apt upgrade -y && \ cd $HOME && \
ver="1.19.1" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```
- Download and install the current mainnet binary version

```
wget https://github.com/likecoin/likecoin-chain/releases/download/v3.0.0/likecoin-chain_3.0.0_Linux_x86_64.tar.gz && \
tar -xvf likecoin-chain_3.1.1_Linux_x86_64.tar.gz && \
chmod +x bin/liked && \
mv bin/liked /usr/local/bin/ && \
rm CHANGELOG.md LICENSE README.md likecoin-chain_3.1.1_Linux_x86_64.tar.gz
```

- Init moniker and set chain-id

```
liked init <MONIKER> --chain-id likecoin-mainnet-2 && \
liked config chain-id likecoin-mainnet-2
```

**Generate keys**
```
liked keys add <wallet_name>
```

**Genesis**
```
curl https://raw.githubusercontent.com/likecoin/mainnet/master/genesis.json > ~/.liked/config/genesis.json
```

**Peers**
```
peers="ccb2af8f2d52324a25ef42a3d2b0b9852675d55b@95.165.89.222:26556"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.liked/config/config.toml
```
**Minimum gas prices**
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"1.0nanolike\"/;" ~/.liked/config/app.toml
```

**Create the service file**
```
sudo tee /etc/systemd/system/liked.service > /dev/null <<EOF
[Unit]
Description=LikeCoin
After=network-online.target

[Service]
User=$USER
ExecStart=$(which liked) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

**Balance**

```
liked q bank balances <you_wallet_address> --node=tcp://localhost:<you_rpc_port_in_config.toml>
```
**Create Validator**

```
liked tx staking create-validator \
  --amount=500000000000nanolike \
  --pubkey=$(liked tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="likecoin-mainnet-2" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="500000000000" \
  --fees="200000nanolike" \
  --from=<wallet_name>
  ```

**Start whith state-sync**

```
sudo systemctl stop liked && liked tendermint unsafe-reset-all --home $HOME/.liked`
curl https://raw.githubusercontent.com/MaxFoton/Likecoin-mainnet/main/addrbook.json > ~/.liked/config/addrbook.json
```

```
SNAP_RPC=95.165.89.222:26556 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```

```
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.liked/config/config.toml
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.liked/config/config.toml
```

**Enable service and start**

```
sudo systemctl daemon-reload && sudo systemctl enable liked
sudo systemctl restart liked && journalctl -fu liked -o cat
```

**Pruning (optional)**

```
pruning="custom" && \
pruning_keep_recent="1000" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.liked/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.liked/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.liked/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.liked/config/app.toml
```

**Indexer (optional)**

```
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.liked/config/config.toml
```

```
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.liked/config/config.toml
```

```
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.liked/config/config.toml
```


```
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.liked/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.liked/config/config.toml
```







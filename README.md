# 0G-Lab-chain-id-16600-1

# Hardware Requirement

    - Memory: 64 GB
    - CPU: 8 cores
    - Disk: 1 TB NVME SSD
    - Bandwidth: 100 MBps for Download / Upload

# Installation guide

Install required packages

    sudo apt update && \
    sudo apt install curl git jq build-essential gcc unzip wget lz4 -y

Install Go

    cd $HOME && \
    ver="1.21.3" && \
    wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
    sudo rm -rf /usr/local/go && \
    sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
    rm "go$ver.linux-amd64.tar.gz" && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
    source $HOME/.bash_profile && \
    go version

Install 0gchaind

    git clone -b v0.1.0 <https://github.com/0glabs/0g-chain.git>
    ./0g-chain/networks/testnet/install.sh
    source .profile

Set Chain ID

    0gchaind config chain-id zgtendermint_16600-1

Initialize Node

    0gchaind init <your_validator_name> --chain-id zgtendermint_16600-1

Download Genesis File

    wget https://raw.githubusercontent.com/Validator247/0G-Lab-chain-id-16600-1/main/genesis.json

Download Addrbook

    wget https://raw.githubusercontent.com/Validator247/0G-Lab-chain-id-16600-1/main/addrbook.json

Seeds & Peers

    PEERS="f878d40c538c8c23653a5b70f615f8dccec6fb9f@54.215.187.94:26656,9d88e34a436ec1b50155175bc6eba89e7a1f0e9a@213.199.61.18:26656"
    SEEDS=""
    sed -i -e "s/^seeds =./seeds = "$SEEDS"/; s/^persistent_peers =./persistent_peers = "$PEERS"/" $HOME/.0gchain/config/config.toml

Create service:

    sudo tee /etc/systemd/system/0gchaind.service > /dev/null <<EOF
    [Unit]
    Description=Ogchain Node
    After=network-online.target

    [Service]
    User=root
    WorkingDirectory=/root/.0gchain
    ExecStart=/root/go/bin/0gchaind start --home /root/.0gchain
    Restart=on-failure
    RestartSec=5
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF

Snapshot

    sudo systemctl stop 0gchaind
    cp $HOME/.0gchain/data/priv_validator_state.json $HOME/.0gchain/priv_validator_state.json.backup
    rm -rf $HOME/.0gchain/data
    0gchaind tendermint unsafe-reset-all --home ~/.0gchain/ --keep-addr-book
    curl https://snapshot.validatorvn.com/og/data.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.0gchain
    mv $HOME/.0gchain/priv_validator_state.json.backup $HOME/.0gchain/data/priv_validator_state.json
    sudo systemctl restart 0gchaind && sudo journalctl -u 0gchaind -f -o cat

State-sync

    sudo systemctl stop 0gchaind
    cp $HOME/.0gchain/data/priv_validator_state.json $HOME/.0gchain/priv_validator_state.json.backup
    0gchaind tendermint unsafe-reset-all --home ~/.0gchain/ --keep-addr-book
    SNAP_RPC="https://ogd-rpc.validatorvn.com:443"

    LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
    BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
    TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
    echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

    sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
    s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
    s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
    s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" ~/.0gchain/config/config.toml
    more ~/.0gchain/config/config.toml | grep 'rpc_servers'
    more ~/.0gchain/config/config.toml | grep 'trust_height'
    more ~/.0gchain/config/config.toml | grep 'trust_hash'

    sudo mv $HOME/.0gchain/priv_validator_state.json.backup $HOME/.0gchain/data/priv_validator_state.json

    sudo systemctl restart 0gchaind && journalctl -u 0gchaind -f -o cat

Create wallet

    0gchaind keys add <key_name> --eth

get your key’s private key

    0gchaind keys unsafe-export-eth-key <key_name>

Create Validator

    0gchaind tx staking create-validator \
    --amount=1700000ua0gi \
    --pubkey=$(0gchaind tendermint show-validator) \
    --moniker="Validator247" \
    --identity="369E23D7BC0B9982" \
    --website="https://validator247.com" \
    --security-contact="validator247@gmail.com" \
    --details="Specializing in operating nodes and validators on EVM and Cosmos ecosystems with Proof-of-Stake" \
    --chain-id=zgtendermint_16600-1 \
    --commission-rate="0.10" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.01" \
    --min-self-delegation=1 \
    --from=wallet \
    --fees=10000ua0gi \
    --gas=300000 \
    -y

# Done         


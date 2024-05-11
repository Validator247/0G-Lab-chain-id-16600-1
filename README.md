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

    git clone -b v0.1.0 https://github.com/0glabs/0g-chain.git
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

Create wallet

    0gchaind keys add wallet --eth

get your key’s private key

    0gchaind keys unsafe-export-eth-key wallet

Convert from Cosmos wallet to evm

    echo "0x$(0gchaind debug addr 0g1dywsndf7xh5n3l690cpgw67j8ekjy9wlp8z8dg | grep hex | awk '{print $3}')"    

Create Validator

    0gchaind tx staking create-validator \
    --amount=1000000ua0gi \
    --pubkey=$(0gchaind tendermint show-validator) \
    --moniker="nodename" \
    --identity="keybase" \
    --website="zzzzzzz" \
    --security-contact="vzzzzzz@gmail.com" \
    --details="love validator247" \
    --chain-id=zgtendermint_16600-1 \
    --commission-rate="0.10" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.01" \
    --min-self-delegation=1 \
    --from=wallet \
    --fees=10000ua0gi \
    --gas=300000 \
    -y

Delegate your validator ( Change the information as appropriate for your node )

    0gchaind tx staking delegate 0gvaloper1yrxeg4symqpfv5r3p9jemkwkgm284j5xgusn6g 980000ua0gi --chain-id=zgtendermint_16600-1 --from wallet --fees=20000ua0gi -y    

# Done         


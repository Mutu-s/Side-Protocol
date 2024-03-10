Update packages & Install dependencies
              
     sudo apt update && sudo apt upgrade -y
     sudo apt install curl build-essential git wget jq make gcc tmux chrony lz4 -y
            
Install GO
            
    sudo rm -rf /usr/local/go
    curl -Ls https://go.dev/dl/go1.19.10.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
    eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
    eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)
              
Download and build binaries

    git clone -b dev https://github.com/sideprotocol/sidechain.git
    cd sidechain
    git checkout 0.0.1-75-gbd63479

make install

    export GOPATH=$HOME/go
    export PATH=$PATH:$GOPATH/bin
              
Initialize Node
              
    sided config chain-id side-testnet-1
    sided init "BccGuide" --chain-id side-testnet-1
    sided config keyring-backend file
            
Download genesis and addrbook
              
    curl -Ls https://services.bccnodes.com/testnets/side/genesis.json > $HOME/.sidechain/config/genesis.json
    curl -Ls https://services.bccnodes.com/testnets/side/addrbook.json > $HOME/.sidechain/config/addrbook.json
              
Config pruning
            
    sed -i \
      -e 's|^pruning *=.*|pruning = "custom"|' \
      -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
      -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
      -e 's|^pruning-interval *=.*|pruning-interval = "10"|' \
      $HOME/.sidechain/config/app.toml

Indexer (optional)
            
    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.sidechain/config/config.toml
            
Set minimum gas price
              
    sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0005uside"|g' $HOME/.sidechain/config/app.toml
            
Enable prometheus
            
    sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.sidechain/config/config.toml
            
Create service
              
    sudo tee /etc/systemd/system/sided.service > /dev/null << EOF
    [Unit]
    Description=sided 
    After=network-online.target
    [Service]
    User=$USER
    ExecStart=$(which sided) start
    Restart=on-failure
    RestartSec=10
    LimitNOFILE=10000
    [Install]
    WantedBy=multi-user.target
    EOF
            
Start service
            
    sudo systemctl daemon-reload
    sudo systemctl enable sided
    sudo systemctl start sided
            
    sudo journalctl -u sided -f


Create a validator
            
    sided tx staking create-validator \
    --amount 1000000uside \
    --commission-max-change-rate "0.1" \
    --commission-max-rate "0.20" \
    --commission-rate "0.1" \
    --min-self-delegation "1" \
    --pubkey=$(sided tendermint show-validator) \
    --moniker BccNodesGuide \
    --chain-id side-testnet-1 \
    --from wallet  

Add New Key
            
    sided keys add wallet
            
Recover existing key
            
    sided keys add wallet --recover

List All Keys
            
    sided keys list

Delete Key
            
    sided keys delete wallet

Export Key (save to wallet.backup)
            
    sided keys export wallet

Import Key
            
    sided keys import wallet wallet.backup

Export Key (save to wallet.backup)
            
    sided q bank balances $(sided keys show wallet -a)

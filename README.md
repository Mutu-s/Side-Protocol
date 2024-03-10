
 > [Discord](https://discord.gg/sideprotocol/)

 > [Testnet](https://testnet.side.one/staking/)

 > [Github](https://github.com/sideprotocol/)

 > [Medium](https://medium.com/@SideProtocol/from-s1-to-s5-side-protocols-path-towards-a-modular-future-e5b7ef137e1a/)


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

 
      peers="9cff0d59c8ca7ac4dd8d641dd6db7d26dfe0e275@173.249.21.127:26656,e77c79648c42f4cbca7e07df53084f73ecac8438@144.126.147.42:656,b2c567c0a698a705ecccab7867632acff4844b2c@38.242.225.98:26656,ccfea7dc3466f74f6a3261d643115f36969c8f3f@65.21.230.12:36656,103321716df75cdf5b8a70cc2fe3ef508de8edb2@161.97.136.24:26656,a3320bcde00a5e1271ecb24e2582c6a393998e02@65.108.233.225:11356,3123a64b713fb712726b0e6035bbca5c143a7989@65.109.67.8:26656,bba20748d8d7b3cb073d9bd9d346d2ca6500ff0f@74.50.67.222:26959,aec730338646650444756599d4cd89cef67f3e23@136.243.9.249:36656,6be4b72d76c4a5e3bb38725323c6b326c9852a81@103.193.175.10:26656,6def6906f05a0d10a671d3cd2005529d320bb3c7@152.228.208.164:26656,3606b846ba099a4a95d1c32f6ab38c966f7362b7@109.199.121.125:26656,aeda846f6aa35c82014e76b461ba7d5bb50ee1a4@109.199.125.110:26656,067ccc939526ec0332e76827afb8b7226fc143b4@167.86.72.144:26656,91a35595954798b0d527425eaa95977d73943a88@45.129.183.252:26656,8e33f96520e74dcd3db95c9d85808400fde39d8b@161.97.74.179:26656,09a972c7f21e16ccc54259edf4b8825725da329a@45.13.59.186:26656,3acdf7ea90e1b3400f41aae803320b3a700db66a@84.247.149.117:26656,b1918afbdb088936466e8488c849ff3e67dc3265@173.249.57.190:26656,264f3b5e3c0481b3cf87239794392a922003b1cb@37.60.227.2:45656"


    sudo systemctl restart sided && sudo journalctl -u sided -f --no-hostname -o cat


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

![242064548-f5e9add1-5b55-40d2-83dd-539cbf64c266](https://github.com/molla202/empower-2/assets/91562185/1191c65e-441f-4d71-9acc-5ea06391b7ed)

<h1 align="center"> Empower Chain </h1>

> Topluluk kanalları: - [Sohbet](https://t.me/corenodechat) - [Empower Discord](https://discord.gg/Zs3GMUhg)

<h1 align="center"> Donanım </h1>


# Sistem
4 CPU
8 RAM
200 SSD
```
# kütüphane kurulumu
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```


```
# install go, if needed
cd $HOME
! [ -x "$(command -v go)" ] && {
VER="1.20.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
}
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin

# set vars ( WALLET VE MONİKER KISMINI DEĞİŞTİRİNİZ )
echo "export WALLET="kriptosekici"" >> $HOME/.bash_profile
echo "export MONIKER="kriptosekici"" >> $HOME/.bash_profile
echo "export EMPOWER_CHAIN_ID="circulus-1"" >> $HOME/.bash_profile
echo "export EMPOWER_PORT="16"" >> $HOME/.bash_profile
source $HOME/.bash_profile

# download binary
cd $HOME
rm -rf empowerchain
git clone https://github.com/EmpowerPlastic/empowerchain
cd empowerchain
git checkout v1.0.0-rc1
cd chain
make install

# config and init app
empowerd config node tcp://localhost:${EMPOWER_PORT}657
empowerd config keyring-backend os
empowerd config chain-id circulus-1
empowerd init "test" --chain-id circulus-1

# download genesis and addrbook
wget -O $HOME/.empowerchain/config/genesis.json https://testnet-files.itrocket.net/empower/genesis.json
wget -O $HOME/.empowerchain/config/addrbook.json https://testnet-files.itrocket.net/empower/addrbook.json

# set seeds and peers
SEEDS="c597ec01e412d6e0f62c6f5501224b7fb8393912@empower-testnet-seed.itrocket.net:16656"
PEERS="c413d3d16e250ddbd8f8d495204b2de46ef36b63@empower-testnet-peer.itrocket.net:16656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.empowerchain/config/config.toml

# set custom ports in app.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${EMPOWER_PORT}317\"%;
s%^address = \":8080\"%address = \":${EMPOWER_PORT}080\"%;
s%^address = \"localhost:9090\"%address = \"0.0.0.0:${EMPOWER_PORT}090\"%; 
s%^address = \"localhost:9091\"%address = \"0.0.0.0:${EMPOWER_PORT}091\"%; 
s%^address = \"localhost:8545\"%address = \"0.0.0.0:${EMPOWER_PORT}545\"%; 
s%^ws-address = \"localhost:8546\"%ws-address = \"0.0.0.0:${EMPOWER_PORT}546\"%" $HOME/.empowerchain/config/app.toml

# set custom ports in config.toml file
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${EMPOWER_PORT}658\"%; 
s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${EMPOWER_PORT}657\"%; 
s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${EMPOWER_PORT}060\"%;
s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${EMPOWER_PORT}656\"%;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${EMPOWER_PORT}656\"%;
s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${EMPOWER_PORT}660\"%" $HOME/.empowerchain/config/config.toml

# config pruning
sed -i -e "s/^pruning *=.*/pruning = \"nothing\"/" $HOME/.empowerchain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.empowerchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.empowerchain/config/app.toml

# set minimum gas price, enable prometheus and disable indexing
sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.0umpwr"/g' $HOME/.empowerchain/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.empowerchain/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.empowerchain/config/config.toml

# create service file
sudo tee /etc/systemd/system/empowerd.service > /dev/null <<EOF
[Unit]
Description=Empower node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.empowerchain
ExecStart=$(which empowerd) start --home $HOME/.empowerchain
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

# reset and download snapshot
empowerd tendermint unsafe-reset-all --home $HOME/.empowerchain
if curl -s --head curl https://testnet-files.itrocket.net/empower/snap_empower.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://testnet-files.itrocket.net/empower/snap_empower.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.empowerchain
    else
  echo no have snap
fi

# enable and start service
sudo systemctl daemon-reload
sudo systemctl enable empowerd
sudo systemctl restart empowerd && sudo journalctl -u empowerd -f -o cat
```

# cüzdan olusturma yada import etme
```
empowerd keys add kriptosekici --recover
```
# cüzdan sorgulama
```
empowerd keys list
```


### Senkronize olmayı bekleyin ardından validatör oluşturun (not: faucetin bir kaç gün içinde açılacağı söylendi)
```
empowerd tx staking create-validator \
  --amount 900000umpwr \
  --from kriptosekici \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(empowerd tendermint show-validator) \
  --moniker kriptosekici \
  --website "websiteniz"
  --identity kriptosekici \
  --details "Rues Community" \
  --chain-id circulus-1
  --y
```
## Restart node
```
sudo systemctl restart empowerd
```
## Log Kontrol
```
journalctl -u empowerd -f -o cat
```
# silme komutları
```
sudo systemctl stop empowerd
sudo systemctl disable empowerd
sudo rm -rf /etc/systemd/system/empowerd.service
sudo rm $(which empowerd)
sudo rm -rf $HOME/.empowerchain
sed -i "/EMPOWER_/d" $HOME/.bash_profile
```
# Sadece port değiştirmek isteyenler

# Port Atama (izmir ayarladım isteyen 35 değiştirsin)
```
echo "export EMPOWERCHAİN_PORT="35"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```
systemctl stop empowerd
```
```
# set custom ports in app.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${EMPOWER_PORT}317\"%;
s%^address = \":8080\"%address = \":${EMPOWER_PORT}080\"%;
s%^address = \"localhost:9090\"%address = \"0.0.0.0:${EMPOWER_PORT}090\"%; 
s%^address = \"localhost:9091\"%address = \"0.0.0.0:${EMPOWER_PORT}091\"%; 
s%^address = \"localhost:8545\"%address = \"0.0.0.0:${EMPOWER_PORT}545\"%; 
s%^ws-address = \"localhost:8546\"%ws-address = \"0.0.0.0:${EMPOWER_PORT}546\"%" $HOME/.empowerchain/config/app.toml
```
```
# set custom ports in config.toml file
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${EMPOWER_PORT}658\"%; 
s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${EMPOWER_PORT}657\"%; 
s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${EMPOWER_PORT}060\"%;
s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${EMPOWER_PORT}656\"%;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${EMPOWER_PORT}656\"%;
s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${EMPOWER_PORT}660\"%" $HOME/.empowerchain/config/config.toml
```
```
sudo systemctl daemon-reload
sudo systemctl restart empowerd
journalctl -u empowerd -f -o cat
```

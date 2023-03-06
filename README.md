# Prepare
- Check explorer ID di https://OSMOSIS.explorers.guru/

__________________________________

# Auto Install
```
curl -sL https://get.osmosis.zone/install > i.py && python3 i.py
```
## Check Log dan Sync pasca Install

### check logs
```
sudo systemctl restart osmosisd && sudo journalctl -u osmosisd -f --no-hostname -o cat
```

### Chek status sync
```
osmosisd status 2>&1 | jq .SyncInfo
```
### check version
```
osmosisd version
```

##### Kalau Pake Auto Install, setelah kelar, langsung lompat ke bagian add wallet gan
__________________________________

# Manual Install
Check di : xxxx
__________________________________
# Add Wallet

### Jika Membuat Wallet baru
```
osmosisd keys add wallet
```

### Jika Import Wallet yang sudah ada
```
osmosisd keys add wallet --recover
```

### Untuk melihat daftar wallet saat ini
```
osmosisd keys list
```
### Simpan Info Wallet
```
OSMOSIS_WALLET_ADDRESS=$(osmosisd keys show $WALLET -a)
OSMOSIS_VALOPER_ADDRESS=$(osmosisd keys show $WALLET --bech val -a)
echo 'export OSMOSIS_WALLET_ADDRESS='${OSMOSIS_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export OSMOSIS_VALOPER_ADDRESS='${OSMOSIS_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
### Faucet 
Discord : https://discord.gg/2J5jw9se4A

### Check Saldo Wallet
```
osmosisd query bank balances $OSMOSIS_WALLET_ADDRESS
```

### Restart Node
```
sudo systemctl daemon-reload
sudo systemctl enable osmosisd
sudo systemctl restart osmosisd
source $HOME/.bash_profile
```
__________________________________
# Validator management
### Create Validator
```
osmosisd tx staking create-validator \
--amount 1000000OSMO \
--pubkey $(osmosisd tendermint show-validator) \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id OSMOSIS-itn-1 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.025OSMO \
-y
```

### Jika ingin Edit existing validator
```
osmosisd tx staking edit-validator \
--new-moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id OSMOSIS-itn-1 \
--from=wallet \
--gas-adjustment 1.4 \
--gas-prices 0.05OSMO
```

__________________________________


# Uninstall Node
```
sudo systemctl stop osmosisd
sudo systemctl disable osmosisd
sudo rm /etc/systemd/system/osmosisd.service
sudo systemctl daemon-reload
rm -f $(which osmosisd)
rm -rf $HOME/.OSMOSIS
rm -rf $HOME/OSMOSIS
rm -rf $HOME/OSMOSIS.sh
rm -rf $HOME/go
```

## Jika Snapshot rusak
```
sudo systemctl stop osmosisd
cp $HOME/.osmosisd/data/priv_validator_state.json $HOME/.osmosisd/priv_validator_state.json.backup
rm -rf $HOME/.osmosisd/data
```
```
curl -L https://snapshots.kjnodes.com/OSMOSIS-testnet/snapshot_latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.osmosisd
mv $HOME/.osmosisd/priv_validator_state.json.backup $HOME/.osmosisd/data/priv_validator_state.json
```
```
sudo systemctl restart osmosisd && sudo journalctl -u osmosisd -f --no-hostname -o cat
```

### Chek status sync
```
osmosisd status 2>&1 | jq .SyncInfo
```
__________________________________

Unjail validator
```
osmosisd tx slashing unjail --from wallet --chain-id OSMOSIS-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025OSMO -y
```

Jail reason
```
osmosisd query slashing signing-info $(osmosisd tendermint show-validator)
```

List all active validators
```
osmosisd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

List all inactive validators
```
osmosisd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

View validator details
```
osmosisd q staking validator $(osmosisd keys show wallet --bech val -a)
```

💲 Token management
Withdraw rewards from all validators
```
osmosisd tx distribution withdraw-all-rewards --from wallet --chain-id OSMOSIS-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.05OSMO -y
```

Withdraw commission and rewards from your validator
```
osmosisd tx distribution withdraw-rewards $(osmosisd keys show wallet --bech val -a) --commission --from wallet --chain-id OSMOSIS-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025OSMO -y
```

Delegate tokens to yourself
```
osmosisd tx staking delegate $(osmosisd keys show wallet --bech val -a) 1000000OSMO --from wallet --chain-id OSMOSIS-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025OSMO -y
```

Delegate tokens to validator
```
osmosisd tx staking delegate <TO_VALOPER_ADDRESS> 1000000OSMO --from wallet --chain-id OSMOSIS-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025OSMO -y
```

Redelegate your stake from one validator to another validator
```
osmosisd tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000OSMO --from wallet --chain-id OSMOSIS-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025OSMO -y
```

Redelegate your stake from another validator to yourself
```
osmosisd tx staking redelegate <srcValidatorAddress> $(osmosisd keys show wallet --bech val -a) 1000000OSMO --from wallet --chain-id OSMOSIS-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025OSMO -y
```

Redelegate your stake from yourself to another validator 
```
osmosisd tx staking redelegate $(osmosisd keys show wallet --bech val -a) <destValidatorAddress> 1000000OSMO --from wallet --chain-id OSMOSIS-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025OSMO -y
```

Unbond tokens from your validator
```
osmosisd tx staking unbond $(osmosisd keys show wallet --bech val -a) 1000000OSMO --from wallet --chain-id OSMOSIS-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025OSMO -y
```

Send tokens to the wallet
```
osmosisd tx bank send wallet <TO_WALLET_ADDRESS> 1000000OSMO --from wallet --chain-id OSMOSIS-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025OSMO -y
```


🗳 Governance
List all proposals
```
osmosisd query gov proposals
```

View proposal by id
```
osmosisd query gov proposal 1
```

Vote 'Yes'
```
osmosisd tx gov vote 1 yes --from wallet --chain-id OSMOSIS-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025OSMO -y
```

Vote 'No'
```
osmosisd tx gov vote 1 no --from wallet --chain-id OSMOSIS-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025OSMO -y
```

Vote 'Abstain'
```
osmosisd tx gov vote 1 abstain --from wallet --chain-id OSMOSIS-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025OSMO -y
```

Vote 'NoWithVeto'
```
osmosisd tx gov vote 1 nowithveto --from wallet --chain-id OSMOSIS-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025OSMO -y
```

⚡️ Utility
Update ports
```
CUSTOM_PORT=10
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}660\"%" $HOME/.osmosisd/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}317\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}091\"%" $HOME/.osmosisd/config/app.toml
```

Update Indexer
Disable indexer
```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.osmosisd/config/config.toml
```

Enable indexer
```
sed -i -e 's|^indexer *=.*|indexer = "kv"|' $HOME/.osmosisd/config/config.toml
```

Update pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.osmosisd/config/app.toml
```

🚨 Maintenance
Get validator info
```
osmosisd status 2>&1 | jq .ValidatorInfo
```

Get sync info
```
osmosisd status 2>&1 | jq .SyncInfo
```

Get node peer
```
echo $(osmosisd tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.osmosisd/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

Check if validator key is correct
```
[[ $(osmosisd q staking validator $(osmosisd keys show wallet --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(osmosisd status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Get live peers
```
curl -sS http://localhost:39657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

Set minimum gas price
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025OSMO\"/" $HOME/.osmosisd/config/app.toml
```

Enable prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.osmosisd/config/config.toml
```

Reset chain data
```
osmosisd tendermint unsafe-reset-all --home $HOME/.osmosisd --keep-addr-book
```

⚙️ Service Management
Reload service configuration
```
sudo systemctl daemon-reload
```

Enable service
```
sudo systemctl enable osmosisd
```

Disable service
```
sudo systemctl disable osmosisd
```

Start service
```
sudo systemctl start osmosisd
```

Stop service
```
sudo systemctl stop osmosisd
```

Restart service
```
sudo systemctl restart osmosisd
```

Check service status
```
sudo systemctl status osmosisd
```

Check service logs
```
sudo journalctl -u osmosisd -f --no-hostname -o cat
```

__________________________________

# Auto Install
```
curl -sL https://get.osmosis.zone/install > i.py && python3 i.py
```
## Check Log dan Sync pasca Install

### check logs
```
sudo systemctl restart osmosisd && sudo journalctl -u osmosisd -f --no-hostname -o cat
```

### Chek sync log setelah 10 menit
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
https://docs.osmosis.zone/networks/join-testnet/
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
--amount 1000000unls \
--pubkey $(osmosisd tendermint show-validator) \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id OSMOSIS-rila \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--fees 675unls \
-y
```

### Submit Crew3
- Check transaksi ID di https://OSMOSIS.explorers.guru/ atau https://explorer-rila.OSMOSIS.io/
- Kemudian copy linknya dan submit di Crew3 : 
https://crew3.xyz/c/OSMOSIS/invite/szl85ZQ5Opq8F_Uj3_siu


### Jika ingin Edit existing validator
```
osmosisd tx staking edit-validator \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL"
--chain-id OSMOSIS-rila \
--commission-rate 0.05 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--fees 675unls \
-y
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
rm -rf $HOME/OSMOSIS-core
rm -rf $HOME/OSMOSIS.sh
rm -rf $HOME/go
```

__________________________________

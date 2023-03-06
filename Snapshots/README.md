# How to Make a Snapshot osmosis

### Configurasi your Domain
Custom CNAME your domain to IP VPS, Like `snapshot.osmosis.example.com`

### Instructions
#### Install dependencies
```
sudo apt update && sudo apt upgrade -y
```

```
sudo apt install nginx certbot python3-certbot-nginx -y
```

```
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
```

```
sudo apt-get update && apt install -y nodejs git
```

```
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | sudo tee /usr/share/keyrings/yarnkey.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
```

```
sudo apt-get update && sudo apt-get install yarn -y
```

### Make Snapshot Folder
```
cd /var/www/
mkdir -p snapshot/osmosis
sudo apt install lz4
```

### Stop Node
```
cd $HOME/.osmosisd
sudo systemctl stop osmosisd
```

### Make Snapshot File
```
tar -cf - data | lz4 > /var/www/snapshot/osmosis/snapshot_latest.tar.lz4
```

### Make Snapshot Config
**Noted** : Please change `<Your_Domain>` to your Snapshot Domain like `snapshot.osmosis.example.com`
```
sudo tee /etc/nginx/sites-enabled/<Your_Domain>.conf >/dev/null <<EOF
server {
        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;
        server_name <Your_Domain>; 


        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                # try_files $uri $uri/ =404;
                root /var/www/snapshot/;
                autoindex on;
    }
}
EOF
```

### Install SSL
```
sudo certbot --nginx --register-unsafely-without-email
```
And select your snapshot domain
        
### Restart Ngin and Node
```
sudo systemctl start nginx
sudo systemctl start osmosisd
```
______________________________
# This Our Snapshot
https://snapshot.osmosis.sarjananode.studio/osmosis/
______________________________
        
## How to Use our Snapshot
### Stop the service and reset the data
```
sudo systemctl stop osmosisd
cp $HOME/.osmosisd/data/priv_validator_state.json $HOME/.osmosisd/priv_validator_state.json.backup
rm -rf $HOME/.osmosisd/data
```
### Download our snapshot
```
curl -L https://snapshot.osmosis.sarjananode.studio/osmosis/snapshot_latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.osmosisd
mv $HOME/.osmosisd/priv_validator_state.json.backup $HOME/.osmosisd/data/priv_validator_state.json
```

### Restart the service and check the log
```
sudo systemctl start osmosisd && sudo journalctl -u osmosisd -f --no-hostname -o cat
```

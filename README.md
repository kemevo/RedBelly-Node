# Telegram
- https://t.me/HerculesNode
# Redbelly node kurulumu (sadece mail gelenler kurabiliyor)
## Sistem gereksinimleri (Rehberde belirtilen gereksinimler ama bence fazla yazmışlar)
 - Ubuntu 22.04+
 - 16 GB RAM
 - 2-4 vCPUs
 - 1 TB storage
 - Portlar : 8545, 8546, 1111, and 1888
 - [Resmi kaynak](https://vine.redbelly.network/nds-node-operating)

# Domain alımı
> Domaini [Namecheap](https://www.namecheap.com/) üzerinden alabilirsiniz. => ornek.xyz
> Dasboard girip aşağıdaki adresimizi ayarlayalım.
> ![redbelly1](https://github.com/kemevo/RedBelly-Node/assets/51703004/fd415dc9-d3f6-4105-b128-030d5761389e)

# Node kurulumu
> Siteden genesis.json, redbelly binary(rbbc olarak yeniden adlandıralım), config_template.yaml (config.yaml olarak yeniden adlandıralım) indirelim ve hepsini ana dizine atalım.

## Güncellemeler ve yükleme
```
sudo apt update
sudo apt install screen snapd net-tools cron curl unzip
sudo DEBIAN_FRONTEND=noninteractive apt-get -y upgrade
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```
```
screen -S red
email=kayıt olduğunuz email
fqn=ornek.xyz ( üstte a record kaydı yaptık ya da redbelly.ornek.xyz nasıl yaptıysanız)
sudo certbot certonly --standalone -d $fqn. --non-interactive --agree-tos -m $email
sudo chown -R $USER:$USER /etc/letsencrypt/
```
> Komutu girdikten sonra ctrl+a+d ile screen den çıkalım.

## Node kaydetme
> [Bu adresten](https://redbelly.atlassian.net/servicedesk/customer/portal/13/group/17/create/86) gerekli bilgileri doldurarak node kaydettiriyoruz
```
Node Url : ornek.xyz
Public address : ödüllerin geleceği adres (alttaki ile farklı olmalı)
Signing address : imzacı adres
Node RPC HTTP port : 8545
Node RPC WS port : 8546
Node consensus port : 1888
Node recovery port : 1111
Discord handle : discord handle
```
> Bu işlemden sonra mail gelmesini bekliyorsunuz. Mail geldikten sonra aşağıdaki işlemleri yapıyorsunuz.

## Config.yaml ayarlama
```
nano config.yaml
```
### Değişecek kısımlar:
- ### ip : ornek.xyz ya da redbelly.ornek.xyz nasıl ayarladıysanız
- ### id : formu doldurduktan sonra mail olarak gönderilecek.
- ### privateKeyHex : formda singing adres olarak verdiğiniz cüzdanın private keyi

## Node başlatma
### Observe.sh oluşturma
```
touch observe.sh
vim observe.sh
```
> Aşadağıki metinde ornek.xyz yazan yerleri kendinize uygun olacak şekilde doldurun. ve kopyalayıp, oberserve.sh için yapıştırın :wq enter ile kaydedip çıkın
```
# filename: observe.sh
if [ ! -d rbn ]; then
  echo rbn doesnt exist. Initialising redbelly
  mkdir -p rbn
  mkdir -p consensus
  cp config.yaml ./consensus

  ./binaries/rbbc init --datadir=rbn --standalone
  rm -rf ./rbn/database/chaindata
  rm -rf ./rbn/database/nodes
  cp genesis.json ./rbn/genesis
else
  echo rbn already exist. continuing with existing setup
  cp config.yaml ./consensus
fi

# Run EVM
rm -f log
./binaries/rbbc run --datadir=rbn --consensus.dir=consensus --tls --consensus.tls --tls.cert=/etc/letsencrypt/live/ornek.xyz/fullchain.pem --tls.key=/etc/letsencrypt/live/ornek.xyz/privkey.pem --http --http.addr=0.0.0.0 --http.corsdomain=* --http.vhosts=* --http.port=8545 --http.api eth,net,web3,rbn --ws --ws.addr=0.0.0.0 --ws.port=8546 --ws.origins="*" --ws.api eth,net,web3,rbn --threshold=200 --timeout=500 --logging.level info --mode production --consensus.type dbft --config.file config.yaml --bootstrap.tries=10 --bootstrap.wait=10 --recovery.tries=10 --recovery.wait=10
```
### Start-rbn.sh oluşturma
```
touch start-rbn.sh
vim start-rbn.sh
```
> Aşadağıki metni kopyalayıp, start-rbn.sh için yapıştırın :wq enter ile kaydedip çıkın
```
#!/bin/sh
# filename: start-rbn.sh
mkdir -p binaries
mkdir -p consensus
chmod +x rbbc
cp rbbc binaries/rbbc
mkdir -p logs
nohup ./observe.sh > ./logs/rbbcLogs 2>&1 &
```

```
chmod +x observe.sh
chmod +x start-rbn.sh
./start-rbn.sh
```
## Loglar
```
tail -f $HOME/logs/rbbcLogs
```
![image](https://github.com/kemevo/RedBelly-Node/assets/51703004/527ddfae-4812-4184-8079-6ff1d1130ed3)

> Discord faucetten token alabilirsiniz imzacı adrese.

## Kontrol
> Metamask ağ ekle diyelim.
- Network name: Redbelly Network
- New RPC URL: (eg. https://ornek.xyz:8545)
- Chain ID: 152
- Currency symbol: RBNT
- Block explorer URL: https://explorer.devnet.redbelly.network/
> Ekle diyelim. Sorun çıkmazsa düzgün çalışıyor demektir.

## Senkron durumu
> Sunucunun bulunduğu blok
```
cat $HOME/logs/rbbcLogs | grep "Imported new chain segment" | tail -1 | awk -F 'number":"' '{print $2}' | cut -d '"' -f 1
```
> Ağın bulunduğu blok
```
echo $(( 16#$(curl -s https://rbn-gcp-australia-southeast1-a-0-b-v2.devnet.redbelly.network:8545/ -X POST -H "Content-Type: application/json" --data '{"method":"eth_getBlockByNumber","params":["latest",false],"id":1,"jsonrpc":"2.0"}' | jq -r .result.number | sed 's/0x//') ))
```
> İkisi eşitse senkron demektir.

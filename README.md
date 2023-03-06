# Tutorial Sao Network testnet Airdrop Finder

<p style="font-size:14px" align="right">
<a href="https://t.me/airdropfind" target="_blank">Join Telegram Airdrop Finder<img src="https://user-images.githubusercontent.com/50621007/183283867-56b4d69f-bc6e-4939-b00a-72aa019d1aea.png" width="30"/></a>
</p>

<p align="center">
  <img height="auto" width="auto" src="https://raw.githubusercontent.com/bayy420-999/airdropfind/main/NavIcon.png">
</p>

## Referensi

[Dokumen resmi](https://docs.sao.network/)

[Validator auction](https://stake-perseverance.chainflip.io/auctions)

[Server Discord Sao Network](https://discord.gg/7n5d9nTSJW)

[Faucet](https://faucet.testnet.sao.network/)

## Persyaratan hardware & software

### Persyaratan hardware

| Komponen | Spesifikasi minimal |
|----------|---------------------|
|CPU|4 Cores|
|RAM|32 GB DDR4 RAM|
|Penyimpanan|1 TB HDD|
|Koneksi|10Mbit/s port|

| Komponen | Spesifikasi rekomendasi |
|----------|---------------------|
|CPU|32 Cores|
|RAM|32 GB DDR4 RAM|
|Penyimpanan|2 x 1 TB NVMe SSD|
|Koneksi|1 Gbit/s port|

### Persyaratan software/OS

| Komponen | Spesifikasi minimal |
|----------|---------------------|
|Sistem Operasi|Ubuntu 16.04|

| Komponen | Spesifikasi rekomendasi |
|----------|---------------------|
|Sistem Operasi|Ubuntu 20.04|

## Dependensi yang dibutuhkan
   * Go versi 1.19.1 keatas
   * wget
   * curl
   * git

* Install wget, curl, git
  ```console
  sudo apt-get update && sudo apt-get upgrade
  apt-get install wget curl git
  ```

* Install Go
  1. Download Go
     ```console
     wget https://go.dev/dl/go1.19.1.linux-amd64.tar.gz
     ```
  2. Install Go
     ```console
     rm -rf /usr/local/go && tar -C /usr/local -xzf go1.19.1.linux-amd64.tar.gz
     ```
  3. Load environment variables
     ```console
     export PATH=$PATH:/usr/local/go/bin
     ```
  4. Cek apakah Go sudah terinstall
     ```console
     go version
     ```

## Task

### Jalankan Consensus Node

1. Download dan build Node
   ```console
   git clone https://github.com/SaoNetwork/sao-consensus.git
   cd sao-consensus
   git checkout testnet0
   make
   ```
2. Join testnet
   ```console
   saod init sao-testnet --chain-id=sao-testnet0
   ```
3. Download config
   ```console
   cd $HOME/.sao/config
   wget https://raw.githubusercontent.com/SAONetwork/sao-consensus/testnet0/network/testnet0/config/app.toml
   wget https://raw.githubusercontent.com/SAONetwork/sao-consensus/testnet0/network/testnet0/config/client.toml
   wget https://raw.githubusercontent.com/SAONetwork/sao-consensus/testnet0/network/testnet0/config/config.toml
   wget https://raw.githubusercontent.com/SAONetwork/sao-consensus/testnet0/network/testnet0/config/genesis.json
   ```

4. Buat daemon
   ```console
   sudo tee /etc/systemd/system/saod.service > /dev/null <<EOF
   [Unit]
   Description=saod
   After=network-online.target
   [Service]
   User=$USER
   ExecStart=$(which saod) start
   Restart=always
   RestartSec=3
   LimitNOFILE=4096
   [Install]
   WantedBy=multi-user.target
   EOF
   ```

5. Jalankan Node
   ```console
   systemctl enable saod && systemctl start saod
   ```

### Jalankan Validator Node

Sebelum lanjut daftar validator, pastikan node kalian sudah sync

Cek status node
```console
saod status | jq
```
Lihat di bagian `catching_up` kalo status masih true berarti belum sync... tunggu dulu sampe sync

1. Buat wallet
   ```console
   saod keys add wallet
   ```
   Masukin passphrase, terus salin mnemonicnya

2. Claim faucet
   https://faucet.testnet.sao.network/

3. Daftar validator
   ```console
   saod tx staking create-validator \
   --amount=10000000sao \
   --pubkey=$(saod tendermint show-validator) \
   --moniker="<NAMA_VALIDATOR> \
   --chain-id=sao-testnet0 \
   --commission-rate="0.10" \
   --commission-max-rate="0.20" \
   --commission-max-change-rate="0.01" \
   --min-self-delegation="1000000" \
   --gas="2000000" \
   --gas-prices="0.0025sao" \
   --from=wallet
   ```

   Lalu cek validatormu
   ```console
   saod query tendermint-validator-set | grep "$(saod tendermint show-address)"
   ```

4. Delegate
   ```console
   saod tx staking delegate $(saod keys show wallet --bech val -a) \
   10000000sao \
   --from=wallet \
   --gas=auto \
   --gas-adjusment=1.5 \
   --gas-prices=0.0025sao
   ```

## Perintah berguna

### Menghentikan Node
```console
systemctl stop saod
```

### Restart Node
```console
systemctl restart saod
```

### Hapus Node
```console
systemctl stop saod && systemctl disable saod
rm -rf sao-consensus .sao
```

### Cek log
```console
journalctl -fu saod -o cat
```

### Cek status Node
```console
saod status | jq
```

### Cek valoper address
```console
saod keys show wallet --bech val -a
```

## Troubleshoot
Reserved

# 🛠️ Warden Node Installation Guide (Chiado Testnet)

This guide walks you through the full setup process for running a **Warden** node on the **Chiado Testnet**, using resources from [Noders Services](https://services.kjnodes.com/testnet/warden/installation/) and [KJNodes](https://github.com/kjnodes).

---

## ⚙️ System Requirements

- OS: Ubuntu 20.04/22.04
- RAM: 4GB minimum (8GB+ recommended)
- Disk: 200GB SSD
- Network: stable internet connection

---

## 🧰 1. Install Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git wget jq make gcc build-essential tmux htop lz4 nano iptables pkg-config libssl-dev libleveldb-dev unzip ncdu
```

---

## 📦 2. Install Go (recommended: 1.22.6)

```bash
cd $HOME
VER="1.22.6"
wget https://go.dev/dl/go$VER.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm go$VER.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bash_profile
source ~/.bash_profile
go version
```

---

## 🔧 3. Set Environment Variables

```bash
echo "export MONIKER=your_moniker" >> ~/.bash_profile
echo "export WARDEN_CHAIN_ID=chiado_10010-1" >> ~/.bash_profile
echo "export WALLET=wallet" >> ~/.bash_profile
source ~/.bash_profile
```

---

## ⬇️ 4. Download Warden Binary

```bash
mkdir -p $HOME/.warden/cosmovisor/genesis/bin
wget -O $HOME/.warden/cosmovisor/genesis/bin/wardend https://github.com/warden-protocol/wardenprotocol/releases/download/v0.6.3/wardend-0.6.3-linux-amd64
chmod +x $HOME/.warden/cosmovisor/genesis/bin/wardend
ln -s $HOME/.warden/cosmovisor/genesis $HOME/.warden/cosmovisor/current -f
sudo ln -s $HOME/.warden/cosmovisor/current/bin/wardend /usr/local/bin/wardend
```

---

## 🔁 5. Install Cosmovisor & Configure Service

```bash
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.6.0
```

Create systemd service:

```bash
sudo tee /etc/systemd/system/warden-testnet.service > /dev/null <<EOF
[Unit]
Description=Warden Chiado Testnet Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.warden"
Environment="DAEMON_NAME=wardend"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/bin:/usr/bin:$HOME/.warden/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable warden-testnet.service
```

---

## 🚀 6. Initialize the Node

```bash
wardend init $MONIKER --chain-id chiado_10010-1
```

Configure client:

```bash
wardend config set client chain-id chiado_10010-1
wardend config set client keyring-backend test
wardend config set client node tcp://localhost:26657
```

---

## 🧬 7. Download Genesis & Addrbook

```bash
curl -Ls https://snapshots.kjnodes.com/warden-testnet/genesis.json > $HOME/.warden/config/genesis.json
curl -Ls https://snapshots.kjnodes.com/warden-testnet/addrbook.json > $HOME/.warden/config/addrbook.json
```

---

## 🛰️ 8. Configure Seeds & Gas Price

```bash
SEEDS="3f472746f46493309650e5a033076689996c8881@warden-testnet.rpc.kjnodes.com:17859"
sed -i -e "s|^seeds *=.*|seeds = \"$SEEDS\"|" $HOME/.warden/config/config.toml

sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "250000000000000award"|' $HOME/.warden/config/app.toml
```

---

## 🌿 9. (Optional) Custom Pruning

```bash
sed -i -e 's/^pruning *=.*/pruning = "custom"/' $HOME/.warden/config/app.toml
sed -i -e 's/^pruning-keep-recent *=.*/pruning-keep-recent = "100"/' $HOME/.warden/config/app.toml
sed -i -e 's/^pruning-interval *=.*/pruning-interval = "19"/' $HOME/.warden/config/app.toml
```

---

## 📦 10. (Optional) Restore Snapshot

```bash
wardend tendermint unsafe-reset-all --home $HOME/.warden
curl -L https://snapshots.kjnodes.com/warden-testnet/snapshot_latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.warden
```

---

## ✅ 11. Start the Node

```bash
sudo systemctl restart warden-testnet.service
sudo journalctl -u warden-testnet.service -f --no-hostname -o cat
```

---

## 🎯 Next Step (Optional): Wallet + Faucet + Validator

You're now syncing! If you'd like to continue and:

- Create a wallet
- Request test tokens
- Become a validator

Let us know, and we’ll walk you through it step-by-step.

---

## 🔗 References

- [DTeam Warden Guide](https://dteam.tech/services/installation-guide/warden/testnet?utm_source=chatgpt.com)
- [KonsorTech Warden Docs](https://konsortech.xyz/testnet/warden/?utm_source=chatgpt.com)
- [KJNodes Installation Guide](https://services.kjnodes.com/testnet/warden/installation/?utm_source=chatgpt.com)
- [Warden GitHub](https://github.com/warden-protocol/wardenprotocol)

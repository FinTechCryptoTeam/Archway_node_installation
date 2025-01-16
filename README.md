# Setting up a validator node for Archway, a Cosmos-based blockchain.



### Ensure System Requirements**
- **CPU:** 4 cores
- **RAM:** 8 GB
- **Disk:** 200 GB SSD
- **Operating System:** Ubuntu 20.04 or 22.04 LTS

---

### Install Dependencies**
Update the system and install necessary tools:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl wget git jq build-essential -y
```

---

### Install Go**
Archway requires Go for building binaries. Install Go:

```bash
wget https://go.dev/dl/go1.21.1.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.21.1.linux-amd64.tar.gz
echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.bashrc
source ~/.bashrc
go version
```

---

### Clone and Build Archway**
Clone the Archway repository and build the binary:

```bash
git clone https://github.com/archway-network/archway.git
cd archway
git checkout v0.3.1 # Replace with the latest stable version if updated
make install
archwayd version
```

---

### Initialize the Node**
Initialize your node with a unique moniker:

```bash
archwayd init "<your_node_name>" --chain-id archway-testnet-1
```

Download the genesis file:

```bash
curl -o ~/.archway/config/genesis.json https://raw.githubusercontent.com/archway-network/networks/main/testnet-1/genesis.json
```

---

### Configure the Node**
Set up persistent peers (replace `<peer_list>` with official peers):

```bash
PEERS="<peer_list>"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ~/.archway/config/config.toml
```

Optimize the configuration:

```bash
# Set minimum gas price
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025uarch\"/" ~/.archway/config/app.toml

# Configure pruning for disk optimization
sed -i.bak -e "s/^pruning *=.*/pruning = \"custom\"/" ~/.archway/config/app.toml
sed -i.bak -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" ~/.archway/config/app.toml
sed -i.bak -e "s/^pruning-interval *=.*/pruning-interval = \"10\"/" ~/.archway/config/app.toml
```

---

### Start the Node**
Create a systemd service for your node:

```bash
sudo tee /etc/systemd/system/archwayd.service > /dev/null <<EOF
[Unit]
Description=Archway Node
After=network.target

[Service]
User=$USER
ExecStart=$(which archwayd) start
Restart=always
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable archwayd
sudo systemctl start archwayd
```

Check the logs:

```bash
journalctl -u archwayd -f
```

---

### Create a Validator**
Once your node is fully synchronized, create a validator:

1. **Fund your wallet:**  
   Transfer testnet tokens to your wallet (from a faucet or another source).

2. **Create the validator:**

   ```bash
   archwayd tx staking create-validator \
     --amount=1000000uarch \
     --pubkey=$(archwayd tendermint show-validator) \
     --moniker="<your_node_name>" \
     --chain-id=archway-testnet-1 \
     --commission-rate="0.10" \
     --commission-max-rate="0.20" \
     --commission-max-change-rate="0.01" \
     --min-self-delegation="1" \
     --from=<YOUR_WALLET> \
     --gas="auto" \
     --fees="1000uarch"
   ```

Replace `<your_node_name>` and `<YOUR_WALLET>` with your actual details.

---

### Monitor Your Node**
Use these commands to monitor your validator:

- **Check node sync status:**
  ```bash
  archwayd status
  ```

- **View validator details:**
  ```bash
  archwayd query staking validator $(archwayd keys show <YOUR_WALLET> --bech val -a)
  ```

- **View logs:**
  ```bash
  journalctl -u archwayd -f
  ```

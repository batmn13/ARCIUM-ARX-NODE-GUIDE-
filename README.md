# ARCIUM-ARX-NODE-GUIDE-
this is a easy guide to run arcium's arx node on you pc 

# 🚀 Arcium Node Setup Guide

a easier way to run arcium's arx node without errors
---

## 📋 Prerequisites

Before you start, make sure you have:

- **Rust** – [Install Rust](https://rustup.rs)
- **Solana CLI** – [Install Solana](https://docs.solana.com/cli/install-solana-cli-tools)
- **Docker & Docker Compose**
- **OpenSSL** (usually preinstalled)
- **Git**
- If you’re on Windows → use **WSL2 (Ubuntu)**

---

## 🧱 Step 1: Create Your Workspace

```
mkdir arcium-node
cd arcium-node
curl https://ipecho.net/plain ; echo  # Get your public IP
```
---

## ⚙️ Step 2: Install Arcium Tooling

```
curl --proto '=https' --tlsv1.2 -sSfL https://install.arcium.com/ | bash

```
---

Verify:

```
arcium --version
arcup --version
````
---

## 🔑 Step 3: Generate Keypairs

```
# Node authority keypair
solana-keygen new --outfile node-keypair.json --no-bip39-passphrase

# Callback authority keypair
solana-keygen new --outfile callback-kp.json --no-bip39-passphrase

# Identity PEM key
openssl genpkey -algorithm Ed25519 -out identity.pem
```
---

## 💰 Step 4: Fund Accounts (Devnet)

```
solana address --keypair node-keypair.json
solana address --keypair callback-kp.json

# Fund with airdrop
solana airdrop 2 <node-pubkey> -u devnet
solana airdrop 2 <callback-pubkey> -u devnet
```
---

## 🧩 Step 5: Initialize Node Accounts

```
solana config set --url https://api.devnet.solana.com

arcium init-arx-accs \
  --keypair-path node-keypair.json \
  --callback-keypair-path callback-kp.json \
  --peer-keypair-path identity.pem \
  --node-offset <your-node-offset> \
  --ip-address <YOUR_PUBLIC_IP> \
  --rpc-url https://api.devnet.solana.com
```
---

## ⚙️ Step 6: Configure Node
Create - node-config.toml

```
[node]
offset = <your-node-offset>
hardware_claim = 0
starting_epoch = 0
ending_epoch = 9223372036854775807

[network]
address = "0.0.0.0"

[solana]
endpoint_rpc = "https://api.devnet.solana.com"
endpoint_wss = "wss://api.devnet.solana.com"
cluster = "Devnet"
commitment.commitment = "confirmed"
```
---

## 🌐 Step 7: Start Your Node (DOCKER RECOMMENDED)

```
mkdir -p arx-node-logs && touch arx-node-logs/arx.log

docker run -d \
  --name arx-node \
  -e NODE_IDENTITY_FILE=/usr/arx-node/node-keys/node_identity.pem \
  -e NODE_KEYPAIR_FILE=/usr/arx-node/node-keys/node_keypair.json \
  -e OPERATOR_KEYPAIR_FILE=/usr/arx-node/node-keys/operator_keypair.json \
  -e CALLBACK_AUTHORITY_KEYPAIR_FILE=/usr/arx-node/node-keys/callback_authority_keypair.json \
  -e NODE_CONFIG_PATH=/usr/arx-node/arx/node_config.toml \
  -v "$(pwd)/node-config.toml:/usr/arx-node/arx/node_config.toml" \
  -v "$(pwd)/node-keypair.json:/usr/arx-node/node-keys/node_keypair.json:ro" \
  -v "$(pwd)/node-keypair.json:/usr/arx-node/node-keys/operator_keypair.json:ro" \
  -v "$(pwd)/callback-kp.json:/usr/arx-node/node-keys/callback_authority_keypair.json:ro" \
  -v "$(pwd)/identity.pem:/usr/arx-node/node-keys/node_identity.pem:ro" \
  -v "$(pwd)/arx-node-logs:/usr/arx-node/logs" \
  -p 8080:8080 \
  arcium/arx-node
```
---

## ✅ Step 8: Verify Node Status

```
arcium arx-info <your-node-offset> --rpc-url https://api.devnet.solana.com
arcium arx-active <your-node-offset> --rpc-url https://api.devnet.solana.com
docker logs -f arx-node
```
---

## 🧠 Tips & Notes to solve errors 
Use unique node offsets to avoid conflicts.

Keep keypairs safe and private.
Open port 8080 if behind a firewall.
Use private RPC providers like Helius or QuickNode for stability.

Your node is live on Arcium testnet
Monitor logs and contribute to the privacy-powered future of computation.

any error you face msg or tag me at 
X- batmn013 , DISCORD- batmn13

#gMPC stay < encrypted >

thanks

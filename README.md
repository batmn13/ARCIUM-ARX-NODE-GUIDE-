# ARCIUM-ARX-NODE-GUIDE-
this is a easy guide to run arcium's arx node on you pc 
run in any error you msg me at my X-@batmn013 and my discord-batmn13

Prerequisites

Rust installed (via rustup.rs)
Solana CLI installed. 
Docker & Docker Compose installed. 
OpenSSL installed (often pre-installed on macOS/Linux). 
Git installed. 
A reliable internet connection + familiarity with command-line. 
If on Windows: Use WSL2 with Ubuntu for this setup.

Step 1: Set Up Your Workspace

mkdir arcium-node-setup
cd arcium-node-setup

Then find your public IP (you’ll need it later):
curl https://ipecho.net/plain ; echo

Step 2: Install Arcium Tooling
Run the installer:
curl --proto '=https' --tlsv1.2 -sSfL https://install.arcium.com/ | bash

This installs:
arcup (Arcium version manager)
Arcium CLI
ARX node software 

Verify installation:
arcium --version
arcup --version

Step 3: Generate Required Keypairs
You’ll generate three keypairs. Run all in your arcium-node-setup directory.

3.1 Node Authority Keypair
solana-keygen new --outfile node-keypair.json --no-bip39-passphrase

3.2 Callback Authority Keypair
solana-keygen new --outfile callback-kp.json --no-bip39-passphrase

3.3 Identity Keypair (PEM format)
openssl genpkey -algorithm Ed25519 -out identity.pem

Security tip: Keep these keypairs safe. Don’t share them

Step 4: Fund Your Accounts
Get the public keys of your keypairs:

solana address --keypair node-keypair.json
solana address --keypair callback-kp.json

Then fund each via Solana Devnet:

# Replace <node-pubkey> with the actual key
solana airdrop 2 <node-pubkey> -u devnet 

# Replace <callback-pubkey> too
solana airdrop 2 <callback-pubkey> -u devnet

If the airdrop fails, you can also use the web faucet: https://faucet.solana.com/

Step 5: Initialize Node Accounts
First, set your Solana CLI to Devnet:

solana config set --url https://api.devnet.solana.com

Then run the account initialization command:

arcium init-arx-accs \
  --keypair-path node-keypair.json \
  --callback-keypair-path callback-kp.json \
  --peer-keypair-path identity.pem \
  --node-offset <your-node-offset> \
  --ip-address <YOUR_PUBLIC_IP> \
  --rpc-url https://api.devnet.solana.com

If success, you’ll see confirmation of your node accounts being created onchain.

Step 6: Configure Your Node
Create a file named node-config.toml in your arcium-node-setup directory with contents such as:

[node]
offset = <your-node-offset>
hardware_claim = 0
starting_epoch = 0
ending_epoch = 9223372036854775807

[network]
address = "0.0.0.0"   # bind to all interfaces

[solana]
endpoint_rpc = "<your-rpc-provider-url-here>"     # e.g., https://api.devnet.solana.com
endpoint_wss = "<your-rpc-websocket-url-here>"    # e.g., wss://api.devnet.solana.com
cluster = "Devnet"
commitment.commitment = "confirmed"              # options: confirmed / processed / finalized

Step 7: Cluster Operations

Your node needs to join a cluster of nodes. You have two options:
Option A: Create Your Own Cluster

arcium init-cluster \
  --keypair-path node-keypair.json \
  --offset <cluster-offset> \
  --max-nodes <max-nodes> \
  --rpc-url https://api.devnet.solana.com

Option B: Join an Existing Cluster
You’ll need an invitation. Then run:

arcium join-cluster true \
   --keypair-path node-keypair.json \
   --node-offset <your-node-offset> \
   --cluster-offset <cluster-offset> \
   --rpc-url https://api.devnet.solana.com

choose true 

Step 8: Deploy Your Node (docker recommended)
8.1 Prepare Log Directory
mkdir -p arx-node-logs && touch arx-node-logs/arx.log

8.2 Start the Container
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

after this run

chmod +x start.sh
./start.sh

Step 9: Check status
arcium arx-info <your-node-offset> --rpc-url https://api.devnet.solana.com

Check if node is active:
arcium arx-active <your-node-offset> --rpc-url https://api.devnet.solana.com

Monitor logs (if using Docker):
docker logs -f arx-node

some notes for you keep track on
STEP 4
faucet link: https://faucet.solana.com/

STEP5 
--keypair-path: your node authority keypair file.
--callback-keypair-path: callback keypair file.
--peer-keypair-path: identity keypair in PEM.
--node-offset: a unique number for your node (8-10 digits recommended). 
docs.arcium.com
--ip-address: your public IP address.
--rpc-url: Devnet endpoint.

STEP6
If you are behind NAT or a firewall, ensure port 8080 is open and forwarded. 
docs.arcium.com

The address = "0.0.0.0" lets the node bind to all local interfaces, while your public IP registered in initialization is how peers connect

STEP7 
(OPTION A)
--offset: unique identifier for the cluster (different from your node offset).
--max-nodes: set how many nodes max in this cluster.

(OPTION B)
  --rpc-url https://api.devnet.solana.com
true means you accept the invite (or false to decline).
--cluster-offset: the cluster’s offset you’re joining.

STEP 8 
The operator keypair uses the same file as your node keypair (simplified for testnet). 
docs.arcium.com

Ensure port 8080 is allowed in firewall/cloud provider. 
docs.arcium.com

You can save this as a script (start.sh) and make it executable:

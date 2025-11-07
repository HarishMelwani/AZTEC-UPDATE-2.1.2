# 🌀 Aztec Validator Migration Guide (v2.1.2)

A simple guide to migrate your Aztec Validator node to the latest **v2.1.2** release.  
Special thanks to [@web3hendrix](https://twitter.com/web3hendrix) and [@lordy2220](https://twitter.com/lordy2220) for making this process simpler ❤️

---

## 🧩 Step 1: Stop Your Node

```bash
cd ~/aztec && docker compose down -v
```

This will stop and remove your running Aztec containers and associated volumes.

---

## ⚙️ Step 2: Install Aztec CLI

```bash
bash -i <(curl -s https://install.aztec.network)
```

Then add it to your PATH:

```bash
echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## 🔧 Step 3: Install Foundry

```bash
curl -L https://foundry.paradigm.xyz | bash
```

Then reload your environment and update Foundry:

```bash
source ~/.bashrc
foundryup
```

---

## 🧠 Step 4: Generate New Validator Keys  
*(Special thanks to [@web3hendrix](https://twitter.com/web3hendrix))*

Create and run this script:

```bash
#!/bin/bash

clear
echo "pls provide your old validator info."
read -sp "   put your OLD Sequencer Private Key (will not be shown): " OLD_PRIVATE_KEY && echo
read -p "   put your sepolia rpc only: " ETH_RPC
echo "Goodd. Starting.." && echo " "

rm -f ~/.aztec/keystore/key1.json
echo ":BE READY to write down your private key both eth and BLS and your eth address."
read -p "   Press [Enter] to generate your new keys..."
aztec validator-keys new --fee-recipient 0x0000000000000000000000000000000000000000000000000000000000000000 && echo " " 
KEYSTORE_FILE=~/.aztec/keystore/key1.json
NEW_ETH_PRIVATE_KEY=$(jq -r '.validators[0].attester.eth' $KEYSTORE_FILE)
NEW_BLS_PRIVATE_KEY=$(jq -r '.validators[0].attester.bls' $KEYSTORE_FILE)
NEW_PUBLIC_ADDRESS=$(cast wallet address $NEW_ETH_PRIVATE_KEY)

echo "good! Your new keys are below. SAVE THIS INFO SECURELY!"
echo "   - NEW ETH Private Key: $NEW_ETH_PRIVATE_KEY"
echo "   - NEW BLS Private Key:  $NEW_BLS_PRIVATE_KEY"
echo "   - NEW Public Address:   $NEW_PUBLIC_ADDRESS"
echo " "

echo "You need to send 0.2 to 0.5 Sepolia eth to this new address:"
echo "   $NEW_PUBLIC_ADDRESS"
read -p "   After the funding tnx is confirmed, press [Enter] to continue.." && echo " "

echo "Approving STAKE spending..."
cast send 0x139d2a7a0881e16332d7D1F8DB383A4507E1Ea7A "approve(address,uint256)" 0xebd99ff0ff6677205509ae73f93d0ca52ac85d67 200000ether --private-key "$OLD_PRIVATE_KEY" --rpc-url "$ETH_RPC" && echo " "

echo "joining the testnet yey..."
aztec add-l1-validator   --l1-rpc-urls "$ETH_RPC"   --network testnet   --private-key "$OLD_PRIVATE_KEY"   --attester "$NEW_PUBLIC_ADDRESS"   --withdrawer "$NEW_PUBLIC_ADDRESS"   --bls-secret-key "$NEW_BLS_PRIVATE_KEY"   --rollup 0xebd99ff0ff6677205509ae73f93d0ca52ac85d67 && echo " "

echo "All done! u have successfully joined the new testnet now rerun your node with your new pvt and address"
```

---

## 🧾 Step 5: Edit Your `.env`

```bash
cd aztec
nano .env
```

Replace the values with your **new private keys** and **addresses** generated in Step 4.

---

## 🧱 Step 6: Edit `docker-compose.yml`

```bash
nano docker-compose.yml
```

Find and replace this line:
```
--snapshots-url https://snapshots.aztec.graphops.xyz/files
```
with:
```
--sync-mode full
```

---

## 🚀 Step 7: Update and Start Your Node

```bash
sed -i 's|image: aztecprotocol/aztec:.*|image: aztecprotocol/aztec:2.1.2|' docker-compose.yml && docker compose pull && docker compose up -d
```

Your node should now be running the **latest version (2.1.2)** 🎉

---

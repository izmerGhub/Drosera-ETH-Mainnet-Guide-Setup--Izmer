# Drosera-ETH-Mainnet-Guide-Setup--Izmer
This guide provides a complete step-by-step process to set up a **Drosera Operator Node** on Ethereum Mainnet, including creating a response trap, configuring `drosera.toml`, registering your operator, and running the node.

---

## Prerequisites

Before you start, ensure you have:

- Linux or WSL (Windows Subsystem for Linux) environment
- Git installed (`sudo apt install git -y`)
- Curl installed (`sudo apt install curl -y`)
- Docker and Docker Compose installed
- At least 0.004 ETH in your wallet for Bloom Blast and operator registration fees
- Your Ethereum private key for transactions
- Your public server IP
- Nano text editor for configuration editing (`sudo apt install nano -y`)

## Table of Contents

1. [Install Drosera CLI](#install-drosera-cli)
2. [Create a Forge Response Trap](#create-a-forge-response-trap)
3. [Configure Drosera TOML](#configure-drosera-toml)
4. [Perform Bloom Blast](#perform-bloom-blast)
5. [Setup Drosera Operator Node](#setup-drosera-operator-node)
6. [Register Operator](#register-operator)
7. [Opt-In Operator to Trap](#opt-in-operator-to-trap)
8. [Run Drosera Node](#run-drosera-node)

---


## 🔹💸 Cheapest VPS Hosting Deals under 3$ per month (Perfect for Testnets & Lightweight Nodes)

Looking for ultra-budget VPS options Here are two solid picks used by many in the blockchain and dev community:

**💰 Budget VPS (Crypto Accepted)**
1. **SpaceCore** - Contabo alternative, accepts crypto
   [CLICK HERE](https://billing.spacecore.pro/billmgr?from=63882)

2. **RackNerd** - From $10.96/year, 30+ crypto options  
   [CLICK HERE](https://my.racknerd.com/aff.php?aff=14994)

3. **HostVDS** - Russian servers, crypto payments  
   [CLICK HERE](https://hostvds.com/?affiliate_uuid=f3d517f2-6e58-4549-9ecd-d280fa8cea3c)

✅ Great for:
- Running lightweight validator or RPC nodes
- Experimenting with testnets
- Hosting low-traffic services

🌍 Suitable for developers on a tight budget or running long-term nodes with minimal cost.


# Drosera Operator Node Setup Guide (Ethereum Mainnet)


## 1. Install Necessary Tools

Install Drosera CLI and operator tools:

```bash
# Install Drosera CLI & tools
curl -L https://app.drosera.io/install | bash

# Reload shell to update PATH
source ~/.bashrc
```

Optional tools:

```bash
# Nano editor (if not installed)
sudo apt update
sudo apt install nano -y

# Git (if needed)
sudo apt install git -y
```

---

## 2. Create a Forge Response Trap

1. Navigate to your working directory:

```bash
mkdir -p ~/my-drosera-trap
cd ~/my-drosera-trap
```

2. Create `NotifySplitsTrap.sol`:

```bash
nano src/NotifySplitsTrap.sol
```

Paste the following code:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ITrap} from "drosera-contracts/interfaces/ITrap.sol";

interface IHarvester {
    function notifySplits(address user, uint256 amount) external;
}

contract NotifySplitsTrap is ITrap {
    address public constant HARVESTER = 0x1975Be6DAD2e829B1cd6C9d618ddD3Ec405C1712;

    function collect() external view returns (bytes memory) {
        return abi.encode(address(0), uint256(0));
    }

    function shouldRespond(bytes[] calldata data) external pure returns (bool, bytes memory) {
        return (true, data[0]);
    }

    function respond(bytes[] calldata data) external {
        (address user, uint256 amount) = abi.decode(data[0], (address, uint256));
        IHarvester(HARVESTER).notifySplits(user, amount);
    }
}
```

3. Build the contract:

```bash
forge build
```

---

## 3. Create `drosera.toml` for Traps

```bash
nano drosera.toml
```

Paste this:

```toml
ethereum_rpc = "USE FREEMIUM RPC TO AVOID ERROR"
drosera_rpc = "https://relay.ethereum.drosera.io/"

[traps]

[traps.notify_splits]
path = "out/NotifySplitsTrap.sol/NotifySplitsTrap.json"
response_contract = "0x1975Be6DAD2e829B1cd6C9d618ddD3Ec405C1712"
response_function = "respond(bytes[])"
cooldown_period_blocks = 33
min_number_of_operators = 1
max_number_of_operators = 2
block_sample_size = 10
private_trap = true
whitelist = ["YOUR WALLET ADDRESS"]

#address will generate after u apply
```

---

## 4. Deploy Trap to Drosera

```bash
# Export private key
export DROSERA_PRIVATE_KEY="YOUR WALLET PRIVATE KEY"

# Apply the trap
drosera apply --eth-rpc-url "FREEMIUM ETH RPC" --private-key $DROSERA_PRIVATE_KEY
```

---

## 5. Create Operator Docker Compose

```bash
mkdir -p ~/Drosera-Network
cd ~/Drosera-Network
nano docker-compose.yml
```

Paste:

```yaml
version: '3.8'

services:
  operator1:
    image: ghcr.io/drosera-network/drosera-operator:latest
    network_mode: host
    command: ["node"]
    environment:
      - DRO__ETH__CHAIN_ID=1
      - DRO__ETH__RPC_URL="FREEMIUM ETH RPC"
      - DRO__ETH__PRIVATE_KEY="YOUR WALLET PRIVATE KEY"
      - DRO__NETWORK__P2P_PORT=31313
      - DRO__NETWORK__EXTERNAL_P2P_ADDRESS="YOUR PUBLIC IP"
      - DRO__DISABLE_DNR_CONFIRMATION=true
      - DRO__SERVER__PORT=31314
      - DRO__INSTRUMENTATION__LOG_LEVEL=debug
      - DRO__INSTRUMENTATION__LOG_FORMAT=full
      - DRO__INSTRUMENTATION__LOG_OUT=stdout
    volumes:
      - op1_data:/data
    restart: always

volumes:
  op1_data:
```

---

## 6. Register Operator

```bash
drosera-operator register \
  --eth-rpc-url "FREEMIUM ETH RPC" \
  --eth-private-key "YOUR WALLET PRIVATE KEY"
```

- This will register your BLS public key with the Drosera registry.

---

## 7. Opt-In Operator to Trap

```bash
drosera opt-in \
  --trap-address "GENERATED TRAP ADDRESS AFTER APPLIED" \
  --eth-rpc-url "FREEMIUM ETH RPC" \
  --eth-private-key "YOUR WALLET PRIVATE KEY"
```

- If using **one operator**, ensure `min_number_of_operators = 1` in TOML.

---

## 8. Run Drosera Operator Node

```bash
cd ~/Drosera-Network
docker compose up -d --force-recreate
docker compose logs -f operator1
```

- Node logs will show:

```
INFO drosera_operator::node: Operator Node successfully spawned!
```

---

## Notes

- Use **separate Ethereum keys** for registration and node operations if restaking is involved.  
- Ensure **ports 31313 (P2P) and 31314 (RPC)** are open in firewall/NAT.  
- Always check **trap deployment** before registering your operator.  

---

## References

- [Drosera Docs](https://docs.drosera.io/)
- [Foundry Forge Docs](https://book.getfoundry.sh/)


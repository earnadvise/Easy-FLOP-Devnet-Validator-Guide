FLOP Network Devnet Validator Setup Guide
<img width="986" height="492" alt="banner" src="https://github.com/user-attachments/assets/88f36241-d87c-4ef6-88af-3a3b855f64f9" />

A complete, beginner-friendly walkthrough for setting up a FLOP Network devnet validator on Windows using WSL2 (Ubuntu).

Preview unavailable

Overview

Flop Labs has launched the FLOP Devnet for testing consensus and data availability infrastructure.

Notice: The FLOP Devnet is an experimental test environment and currently does not award tokens or airdrop allocations. Use this guide to learn the setup process and prepare your validator keys for the upcoming public testnet.

Setup Requirements
Component	Requirement	Cost
Operating System	Windows 10/11 (with WSL2 Ubuntu) or Ubuntu Linux	Free
Hardware	8 GB+ RAM, 50 GB free disk space	Free
Validator Uptime	20–30 minutes for identity & key generation	No 24/7 required
Devnet Tokens	Provided via official faucet and operations funding	Free
Step 1: Install Ubuntu via WSL2

If you do not have WSL2 installed on Windows:

Open PowerShell as Administrator (Right-click PowerShell -> Run as administrator).
Run the installation command:
powershell
wsl --install -d Ubuntu-22.04
Restart your computer if prompted by Windows.
Launch Ubuntu 22.04 from the Start menu and set a username and password.
Step 2: Install Base Dependencies

Open your Ubuntu terminal and run the commands below to install Docker, Node.js 24, and pnpm:

bash
# Update package repositories and install build tools
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget git jq build-essential ca-certificates gnupg lsb-release age
# Install Docker engine
sudo apt install -y docker.io docker-compose-v2
sudo usermod -aG docker $USER
sudo service docker start
# Install Node.js 24
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt-get install -y nodejs
# Enable pnpm package manager
sudo corepack enable
sudo corepack prepare pnpm@12.14.0 --activate

Verify your environment versions:

bash
node -v      # v24.x
pnpm -v      # 12.14.0
docker -v    # Docker version 24+
Step 3: Download & Extract Validator Package

Download the official devnet release package (validator-cs-be91fe83):

bash
mkdir -p ~/flop && cd ~/flop
# Download package and checksum
wget https://explorer.devnet.flop.network/validator-downloads/validator-cs-be91fe83-0.3.0-devnet.1.tar.gz
wget https://explorer.devnet.flop.network/validator-downloads/validator-cs-be91fe83-0.3.0-devnet.1.tar.gz.sha256
# Verify package integrity
sha256sum -c validator-cs-be91fe83-0.3.0-devnet.1.tar.gz.sha256
# Extract files
tar -xzf validator-cs-be91fe83-0.3.0-devnet.1.tar.gz
cd validator-cs-be91fe83
Step 4: Configure Host & Generate Keys

Set your hostname and configure /etc/hosts:

bash
sudo hostnamectl set-hostname myvalidator
echo "127.0.0.1 $(hostname)" | sudo tee -a /etc/hosts

Generate the host SSH key (required for encrypted Tailscale network onboarding):

bash
sudo ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N ""

Install project dependencies and execute onboarding:

bash
pnpm install --frozen-lockfile --ignore-scripts
pnpm onboard prepare

The script will generate your local private configuration (account.json and .env) and print your Public SS58 Address.

Step 5: Format & Share Public Onboarding Values

Run the script below to collect the three public values needed by operations:

bash
mkdir -p ~/flop
PUBLIC_NOTE=~/flop/ops-public-values.txt
{
  echo "Hostname: $(hostname -f 2>/dev/null || hostname)"
  echo "Public SS58 address: $(jq -r '.address // .ss58Address // .publicAddress // empty' account.json 2>/dev/null || true)"
  echo "Host SSH public key:"
  cat /etc/ssh/ssh_host_ed25519_key.pub 2>/dev/null || true
  echo
  echo "Host SSH fingerprint:"
  ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub 2>/dev/null || true
} | tee "$PUBLIC_NOTE"
Submission Template

Send only these three public values to your FLOP operations contact or comment on flop-labs/network#10
:

text
Hi FLOP Operations Team,
I have completed the local validator preparation (`pnpm onboard prepare`) using package `validator-cs-be91fe83 · 0.3.0-devnet.1` and would like to request enrollment into the devnet validator set.
Here are my public onboarding values:
- Hostname: myvalidator
- Public SS58 Address: <YOUR_PUBLIC_SS58_ADDRESS>
- Host SSH Public Key: <YOUR_SSH_PUBLIC_KEY>
Looking forward to receiving the Tailscale enrollment key, stake funding, and da-peers.json to execute `pnpm onboard join`. Thank you!

Security Reminder: Never share your private key, seed phrase, account.json, or .env file publicly. Only provide the public values listed above.

Step 6: Wait for Operations Approval & Join

At this point, you can safely close your terminal and turn off your PC. You do not need to keep the node running.

Devnet validator onboarding is administrator-assisted. Once operations approves your submission, they will supply:

Encrypted Tailscale network credentials.
Devnet stake funding to your SS58 address.
The da-peers.json configuration file.

Once received, place da-peers.json into ~/flop/validator-cs-be91fe83 and run:

bash
cd ~/flop/validator-cs-be91fe83
pnpm onboard join
pnpm onboard verify
Official Resources
Operator Documentation: https://docs.flop.network/operators
Devnet Explorer: https://explorer.devnet.flop.network
Devnet Faucet: https://faucet.flop.network
GitHub Network Repository: https://github.com/flop-labs/network

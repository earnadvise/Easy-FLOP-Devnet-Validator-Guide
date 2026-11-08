Flop Network Devnet Validator Setup Guide

A simple, practical guide to setting up a FLOP Network devnet validator on your home PC using WSL2 (Ubuntu) on Windows.

Preview unavailable

Important Notes Before You Start
The devnet is currently disposable. Flop Labs has made it clear that running on devnet does not earn token rewards or airdrop points.
Run this if you want to test the setup early and be ready for the public testnet.
You do not need to keep your PC running 24/7. The initial setup only takes about 20 to 30 minutes to generate your keys and identity.
Never share your 12-word recovery phrase, account.json, or .env file with anyone. Only share your public SS58 address and SSH public key.
Requirements
A Windows 10 or 11 PC (with WSL2 enabled) or any standard Ubuntu 22.04 / 24.04 Linux machine.
Around 50 GB free disk space and 8 GB RAM for the preparation phase.
Total cost: $0.
Step 1: Set Up WSL2 on Windows

If you don't have Ubuntu on Windows yet:

Open PowerShell as Administrator.
Run:
powershell
wsl --install -d Ubuntu-22.04
Restart your PC if prompted, then open Ubuntu from your Start menu and set a username and password.
Step 2: Install Required Dependencies

Inside your Ubuntu terminal, run the following commands to install Docker, Node.js 24, and pnpm:

bash
# Update packages and install essentials
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget git jq build-essential ca-certificates gnupg lsb-release age
# Install Docker
sudo apt install -y docker.io docker-compose-v2
sudo usermod -aG docker $USER
sudo service docker start
# Install Node.js 24
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt-get install -y nodejs
# Enable pnpm
sudo corepack enable
sudo corepack prepare pnpm@12.14.0 --activate

You can confirm everything is installed with:

bash
node -v
pnpm -v
docker -v
Step 3: Download the Validator Package
bash
mkdir -p ~/flop && cd ~/flop
# Download the latest package and sha256 checksum
wget https://explorer.devnet.flop.network/validator-downloads/validator-cs-be91fe83-0.3.0-devnet.1.tar.gz
wget https://explorer.devnet.flop.network/validator-downloads/validator-cs-be91fe83-0.3.0-devnet.1.tar.gz.sha256
# Verify the file
sha256sum -c validator-cs-be91fe83-0.3.0-devnet.1.tar.gz.sha256
# Extract
tar -xzf validator-cs-be91fe83-0.3.0-devnet.1.tar.gz
cd validator-cs-be91fe83
Step 4: Configure Host and Run Prepare

Set your hostname and avoid WSL local host errors:

bash
sudo hostnamectl set-hostname myvalidator
echo "127.0.0.1 $(hostname)" | sudo tee -a /etc/hosts

Generate an SSH host key (operations uses this to encrypt your Tailscale network access):

bash
sudo ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N ""

Install package dependencies and run the onboard prepare script:

bash
pnpm install --frozen-lockfile --ignore-scripts
pnpm onboard prepare

When finished, this command creates your private identity files (account.json and .env) and prints your public SS58 address on screen.

Step 5: Get Your Public Values for Operations

To view the exact details you need to send to the FLOP team:

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

Submit these three public items to the FLOP team coordinator or leave a comment on GitHub issue flop-labs/network#10:

Hostname
Public SS58 Address
Host SSH Public Key

Note: Never send account.json, .env, or your seed phrase. Only send the public values listed above.

Step 6: Next Steps (Joining the Network)

At this point, you can shut down your PC. You do not need to keep the node running right now.

The FLOP devnet validator onboarding is currently administrator-assisted. Once operations approves your details, they will provide:

A Tailscale network enrollment key.
Devnet stake funding to your SS58 address.
The da-peers.json file.

After receiving those, place da-peers.json in ~/flop/validator-cs-be91fe83 and run:

bash
cd ~/flop/validator-cs-be91fe83
pnpm onboard join
pnpm onboard verify
Useful Links
Official Docs: https://docs.flop.network
Devnet Explorer: https://explorer.devnet.flop.network
Faucet: https://faucet.flop.network
FLOP Network GitHub: https://github.com/flop-labs/network

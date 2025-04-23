# SeismicWalnutApp

# This is not an official guide. It's a shortened guide for quickly getting started without unnecessary information. It's a shortened guide for quickly getting started without unnecessary information. The  full guide with all explanations is available here:
https://docs.seismic.systems/onboarding/tutorial
My X - https://x.com/maximkapk

# Here are the steps how you can interact with Seismic technology on your computer, just follow steps and see how you slightly become a developer.

# For local development you should have this things on your PC. 
# sforge - testing framework, sanvil - local node, ssolc - compiler. Also, some things as Rust, Cargo, Brew, JQ and Bun have to be installed. 

# install rust and cargo.
curl https://sh.rustup.rs -sSf | sh

# Download and execute the sfoundryup installation script.
curl -L \
     -H "Accept: application/vnd.github.v3.raw" \
     "https://api.github.com/repos/SeismicSystems/seismic-foundry/contents/sfoundryup/install?ref=seismic" | bash
source ~/.zshenv  # or ~/.bashrc or ~/.zshrc

# Install sforge, sanvil, ssolc. Expect this to take between 5-20 minutes depending on your machine.
sfoundryup
source ~/.zshenv  # or ~/.bashrc or ~/.zshrc

# Remove old build artifacts in existing projects.
sforge clean  # run in your project's contract directory

# Also install Seismic extension in VScode if you don't have it 
https://marketplace.visualstudio.com/items?itemName=SeismicSys.seismic

# Install bun 
Mac&Linux - curl -fsSL https://bun.sh/install | bash
Windows - powershell -c "irm bun.sh/install.ps1 | iex"

# Check the version of sanvil, sforge and ssolc 
sanvil --version

# Open you code editor and terminal. Create the project folder and navigate into it:
mkdir walnut-app
cd walnut-app

# Create the packages directory with subdirectories for contracts and cli
mkdir -p packages/contracts packages/cli

# Initialize a bun project in the root directory:
bun init -y && rm index.ts && rm tsconfig.json && touch .prettierrc && touch .gitmodules

# Replace the default package.json with the following content for a monorepo setup:
{
  "workspaces": [
    "packages/**"
  ],
  "dependencies": {},
  "devDependencies": {
    "@trivago/prettier-plugin-sort-imports": "^5.2.1",
    "prettier": "^3.4.2"
  }
}

# Add the code from the .prettierrc to your .prettierrc file for consistent code formatting:

# Replace the.gitignore file with:
cache/
out/
!/broadcast
/broadcast/*/31337/
/broadcast/**/dry-run/
docs/
.env
node_modules/

# Add the following to the .gitmodules file to track git submodules (in our case, only the Forge standard library, forge-std):
[submodule "packages/contracts/lib/forge-std"]
	path = packages/contracts/lib/forge-std
	url = https://github.com/foundry-rs/forge-std

# Navigate to the contracts subdirectory:
cd packages/contracts

# Initialize a project with sforge: 
sforge init --no-commit && rm -rf .github

# Edit the .gitignore file to be the following:
.env
broadcast/
cache/

# Delete the default contract, test and script files (Counter.sol and Counter.t.sol  and Counter.s.sol ) and replace them with their Walnut counterparts (Walnut.sol , Walnut.t.sol  and Walnut.s.sol ):
rm -f src/Counter.sol test/Counter.t.sol script/Counter.s.sol

# Create empty Walnut files in the same locations
touch src/Walnut.sol test/Walnut.t.sol script/Walnut.s.sol

# Navigate to the contracts subdirectory:
# Assuming you are currently in the contracts directory
cd ../cli

# Initialize a new bun project
bun init -y

# Now, create an src/ folder and move index.ts there.
mkdir -p src && mv -t src index.ts 
# For macOS
mkdir -p src && mv index.ts src/ 

# Edit package.json with the code from package.json

# Edit .gitignore
node_modules

# Open up packages/contracts/Walnut.sol and paste the code from Walnut.sol 

# Navigate to the test folder in your Walnut App and open the Walnut.t.sol file located at:
packages/contracts/test/Walnut.t.sol
# open Walnut.t.sol and paste code from Walnut.t.sol file

# Test out the file by running the following inside the packages/contracts directory:
sforge build
sforge test

# Navigate to the script folder in your Walnut App and open the Walnut.s.sol file located at:
packages/contracts/script
# and add code from Walnut.s.sol

# For deploying the contract. In a separate terminal window, run: 
sanvil

# In packages/contracts , create a .env file and add the following to it:
RPC_URL=http://127.0.0.1:8545
PRIVKEY=0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80

# Now, from packages/contracts, run
source .env
sforge script script/Walnut.s.sol:WalnutScript \
    --rpc-url $RPC_URL \
    --broadcast

# Your contract should be up and deployed to your local Seismic node!
    












































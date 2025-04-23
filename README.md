# SeismicWalnutApp|
-------------------

# Here are the steps how you can interact with Seismic technology on your computer, just follow steps and see how you slightly become a developer.

# For local development you should have this things on your PC. 
# sforge - testing framework
# sanvil - local node
# ssolc - compiler

# Also, some things as Rust, Cargo, Brew, JQ and Bun have to be installed. 

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

# Open you code editor and terminal
# Create the project folder and navigate into it:
mkdir walnut-app
cd walnut-app

# Create the packages  directory with subdirectories for contracts and cli
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

# Add the following to the .prettierrc file for consistent code formatting:
{
  "semi": false,
  "tabWidth": 2,
  "singleQuote": true,
  "printWidth": 80,
  "trailingComma": "es5",
  "plugins": ["@trivago/prettier-plugin-sort-imports"],
  "importOrder": [
    "<TYPES>^(?!@)([^.].*$)</TYPES>",
    "<TYPES>^@(.*)$</TYPES>",
    "<TYPES>^[./]</TYPES>",
    "^(?!@)([^.].*$)",
    "^@(.*)$",
    "^[./]"
  ],
  "importOrderParserPlugins": ["typescript", "jsx", "decorators-legacy"],
  "importOrderSeparation": true,
  "importOrderSortSpecifiers": true
}


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


# Edit package.json 
{
    "name": "walnut-cli",
    "license": "MIT License",
    "type": "module",
    "scripts": {
        "dev": "bun run src/index.ts"
    },
    "dependencies": {
        "dotenv": "^16.4.7",
        "seismic-viem": "1.0.9",
        "viem": "^2.22.3"
    },
    "devDependencies": {
        "@types/node": "^22.7.6",
        "typescript": "^5.6.3"
    }
}

# Edit .gitignore
node_modules

# Open up packages/contracts/Walnut.sol and paste the code below

// SPDX-License-Identifier: MIT License
pragma solidity ^0.8.13;

contract Walnut {
    uint256 initialShellStrength; // The initial shell strength for resets.
    uint256 shellStrength; // The current shell strength.
    uint256 round; // The current round number.

    suint256 initialKernel; // The initial hidden kernel value for resets.
    suint256 kernel; // The current hidden kernel value.

    // Tracks the number of hits per player per round.
    mapping(uint256 => mapping(address => uint256)) hitsPerRound;

    // Events to log hits, shakes, and resets.

    // Event to log hits.
    event Hit(uint256 indexed round, address indexed hitter, uint256 remaining);
    // Event to log shakes.
    event Shake(uint256 indexed round, address indexed shaker);
    // Event to log resets.
    event Reset(uint256 indexed newRound, uint256 shellStrength);

    constructor(uint256 _shellStrength, suint256 _kernel) {
        initialShellStrength = _shellStrength; // Set the initial shell strength.
        shellStrength = _shellStrength; // Initialize the shell strength.

        initialKernel = _kernel; // Set the initial kernel value.
        kernel = _kernel; // Initialize the kernel value.

        round = 1; // Start with the first round.
    }

    // Get the current shell strength.
    function getShellStrength() public view returns (uint256) {
        return shellStrength;
    }

    // Hit the Walnut to reduce its shell strength.
    function hit() public requireIntact {
        shellStrength--; // Decrease the shell strength.
        hitsPerRound[round][msg.sender]++; // Record the player's hit for the current round.
        emit Hit(round, msg.sender, shellStrength); // Log the hit.
    }

    // Shake the Walnut to increase the kernel value.
    function shake(suint256 _numShakes) public requireIntact {
        kernel += _numShakes; // Increment the kernel value.
        emit Shake(round, msg.sender); // Log the shake.
    }

    // Reset the Walnut for a new round.
    function reset() public requireCracked {
        shellStrength = initialShellStrength; // Reset the shell strength.
        kernel = initialKernel; // Reset the kernel value.
        round++; // Move to the next round.
        emit Reset(round, shellStrength); // Log the reset.
    }

    // Look at the kernel if the shell is cracked and the caller contributed.
    function look() public view requireCracked onlyContributor returns (uint256) {
        return uint256(kernel); // Return the kernel value.
    }

    // Set the kernel to a specific value.
    function set_number(suint _kernel) public {
        kernel = _kernel;
    }

    // Modifier to ensure the shell is fully cracked.
    modifier requireCracked() {
        require(shellStrength == 0, "SHELL_INTACT");
        _;
    }

    // Modifier to ensure the shell is not cracked.
    modifier requireIntact() {
        require(shellStrength > 0, "SHELL_ALREADY_CRACKED");
        _;
    }

    // Modifier to ensure the caller has contributed in the current round.
    modifier onlyContributor() {
        require(hitsPerRound[round][msg.sender] > 0, "NOT_A_CONTRIBUTOR");
        _;
    }
}

# Navigate to the test folder in your Walnut App and open the Walnut.t.sol file located at:
packages/contracts/test/Walnut.t.sol
# open Walnut.t.sol and paste code from Walnut.t.sol file

# Test out the file by running the following inside the packages/contracts directory:
sforge build
sforge test

# Navigate to the script folder in your Walnut App and open the Walnut.s.sol file located at:
packages/contracts/script

# and add the following to it:
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import {Script, console} from "forge-std/Script.sol";
import {Walnut} from "../src/Walnut.sol";

contract WalnutScript is Script {
    Walnut public walnut;

    function run() public {
        uint256 deployerPrivateKey = vm.envUint("PRIVKEY");

        vm.startBroadcast(deployerPrivateKey);
        walnut = new Walnut(3, suint256(0));
        vm.stopBroadcast();
    }
}

# Deploying the contract 
# In a separate terminal window, run: 
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
    












































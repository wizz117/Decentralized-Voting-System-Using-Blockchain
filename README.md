# Decentralized Voting System Using Blockchain

A full-stack proof-of-concept that demonstrates how **Ethereum smart contracts** can power a tamper-resistant voting platform while a traditional **MySQL** backend stores voter metadata.  
The stack looks like this:

| Layer | Tech | Purpose |
|-------|------|---------|
| **Smart-Contract** | Solidity + Truffle | Ballot creation, casting & tallying |
| **Blockchain Node** | Ganache (local) | Personal Ethereum network for dev/testing |
| **Backend API** | FastAPI + MySQL | Voter registry & auth tokens |
| **Frontend** | Vanilla JS + Browserify | Lightweight UI that talks to both layers |
| **Wallet** | MetaMask | Signs transactions & switches accounts |

---

## Features

* **One-person-one-vote** enforcement on-chain  
* **End-to-end verifiability** – anyone can audit the block data
* **JWT-based voter auth** backed by MySQL
* **Hot-reloading dev servers** for both Python API and Node static site
* **Modular contract migrations** with Truffle

---

## Table of Contents

1. [Prerequisites](#-prerequisites)
2. [Quick Start](#-quick-start)
3. [Detailed Setup](#-detailed-setup)
4. [Project Structure](#-project-structure)
5. [Usage](#-usage)
6. [Contributing](#-contributing)

---

## Prerequisites

| Tool | Tested Version |
|------|----------------|
| Node.js | ≥ 18.x |
| npm | ≥ 9.x |
| Python | ≥ 3.10 |
| MySQL | ≥ 8.0 |
| Ganache | 2.7 (GUI) |
| MetaMask | Latest |

---

## Quick Start

```bash
# 1 ╱ Clone repo & install JS deps
git clone https://github.com/<your-user>/Decentralized-Voting-System-Using-Blockchain.git
cd Decentralized-Voting-System-Using-Blockchain
npm install

# (If you see @babel.zip or abortcontroller-polyfill.zip)
unzip @babel.zip -d node_modules/
unzip abortcontroller-polyfill.zip -d node_modules/

# 2 ╱ Install global tools
npm  install -g truffle           # Solidity framework
pip install --user 'fastapi[all]' mysql-connector-python python-dotenv PyJWT

# 3 ╱ Spin up Ganache & MetaMask
# - open Ganache GUI → new workspace → import ./truffle-config.js
# - in MetaMask, add “Localhost 7575” network (RPC URL http://localhost:7545 / Chain ID 1337)
# - import the first few Ganache accounts into MetaMask

# 4 ╱ Configure MySQL
mysql -u root -p
> CREATE DATABASE voter_db;
> USE voter_db;
> CREATE TABLE voters (id INT PRIMARY KEY AUTO_INCREMENT, eth_address VARCHAR(42) UNIQUE, name VARCHAR(60));

# 5 ╱ Put DB creds into ./Database_API/.env
echo "MYSQL_USER=root
MYSQL_PASS=...
MYSQL_HOST=127.0.0.1
MYSQL_DB=voter_db
JWT_SECRET=$(openssl rand -hex 32)" > Database_API/.env

# 6 ╱ Compile & migrate contracts
truffle compile
truffle migrate --network development   # Ganache must be running

# 7 ╱ Run backend API
cd Database_API
uvicorn main:app --reload --host 127.0.0.1 &
cd ..

# 8 ╱ Bundle frontend & launch node server
browserify ./src/js/app.js -o ./src/dist/app.bundle.js
node index.js

# 9 ╱ Open http://localhost:8080  🎉
```
## Detailed Setup
1. Ganache Workspace
- Start Ganache GUI → New Workspace
- Point workspace to this repo’s truffle-config.js so Truffle & Ganache share the same chain ID (1337) and port (7545).

2. MetaMask
- Install the extension → Create Wallet
- Import the first Ganache test account (private key in Ganache UI).
- Add Network →
- Name: Localhost 7575
- RPC URL: http://localhost:7545
- Chain ID: 1337
- Currency Symbol: ETH

3. MySQL Schema
- Feel free to extend the starter table:
```bash
CREATE TABLE voters (
  id          INT AUTO_INCREMENT PRIMARY KEY,
  eth_address VARCHAR(42) NOT NULL UNIQUE,
  name        VARCHAR(60),
  has_voted   BOOLEAN DEFAULT 0,
  created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
4. Environment Variables (Database_API/.env)
```bash
MYSQL_HOST=127.0.0.1
MYSQL_DB=voter_db
MYSQL_USER=root
MYSQL_PASS=********
JWT_SECRET=<random-32-byte-hex>
```
5. Contract Scripts
- contracts/Voting.sol – core ballot logic
- migrations/2_deploy_voting.js – deploys the contract with initial candidates

## Project Structure
```bash
Decentralized-Voting-System-Using-Blockchain/
├── contracts/              # Solidity smart contracts
├── migrations/             # Truffle deployment scripts
├── build/contracts/        # ABI & bytecode (auto-generated)
├── src/                    # Front-end (JS/CSS/HTML)
│   ├── js/                 #   raw ES6 modules
│   └── dist/               #   Browserify bundle output
├── Database_API/           # FastAPI + MySQL backend
│   ├── main.py
│   └── .env                #   DB creds & JWT secret
├── index.js                # Express static server (port 8080)
└── truffle-config.js       # Truffle & Ganache settings
```
## Usage
- Register Voter – submit your Eth address & name (stored in MySQL).
- Create Election – (owner only) deploys new ballot contract.
- Cast Vote – MetaMask pops up to sign the TX; duplicate votes are rejected on-chain.
- See Results – tallies come straight from the contract’s public state.

## Contributing
- Pull requests are welcome! Please open an issue first to discuss major changes.
- Make sure your code passes npm run lint and truffle test before submitting.

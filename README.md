# TP1 — Blockchain & Développement — ERC-20 from scratch

Token ERC-20 implémenté depuis zéro, avec tests unitaires Hardhat (TypeScript) et Foundry (Solidity/fuzzing), profiling gas et déploiement sur Sepolia.

## Prérequis

- [Node.js v20+ LTS](https://nodejs.org/en/download)
- [Git](https://git-scm.com/downloads)
- [VS Code](https://code.visualstudio.com) (recommandé) + extension Hardhat
- Extension [Metamask](https://metamask.io) dans le navigateur

## Installation

### 1. Foundry

```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup

# Vérification
forge --version
cast --version
anvil --version
```

### 2. Faucets ETH Sepolia

De l'ETH de test est nécessaire pour déployer sur Sepolia :

- [Alchemy Sepolia Faucet](https://sepoliafaucet.com) (recommandé, nécessite un compte Alchemy)
- [Infura Sepolia Faucet](https://www.infura.io/faucet/sepolia)
- [Google Cloud Web3 Faucet](https://cloud.google.com/application/web3/faucet/ethereum/sepolia)

### 3. Clé API RPC

Créer un compte gratuit sur [Alchemy](https://alchemy.com) et une application Sepolia. Noter l'URL RPC :

```
https://eth-sepolia.g.alchemy.com/v2/VOTRE_CLE_API
```

## Initialisation du workspace

### Projet Hardhat

```bash
mkdir tp1-erc20 && cd tp1-erc20
npm init -y
npm install --save-dev hardhat
npx hardhat init
# → Choisir : TypeScript project
# → Oui pour .gitignore et installer les dépendances

npm install --save-dev @nomicfoundation/hardhat-toolbox
npm install --save-dev @openzeppelin/contracts
npm install --save-dev hardhat-gas-reporter
npm install --save-dev dotenv
```

### Configuration `hardhat.config.ts`

Voir le fichier `hardhat.config.ts` du dépôt (réseau `sepolia`, gas reporter en EUR, optimiseur activé avec `viaIR`).

### Fichier `.env`

Créer un fichier `.env` à la racine (⚠️ ne jamais le commiter — vérifier qu'il figure bien dans `.gitignore`, et utiliser un wallet dédié au développement) :

```bash
RPC_URL_SEPOLIA=https://eth-sepolia.g.alchemy.com/v2/VOTRE_CLE
PRIVATE_KEY=0xVOTRE_CLE_PRIVEE_WALLET_TEST
ETHERSCAN_API_KEY=VOTRE_CLE_ETHERSCAN
```

### Foundry dans le même workspace

```bash
forge init --force --no-git .
rm -f contracts/Counter.sol test/Counter.t.sol script/Counter.s.sol
forge build   # doit compiler sans erreur
```

Vérifier que `foundry.toml` pointe `src` vers `contracts/` et contient les remappings (`@openzeppelin/`, `forge-std/`, etc. selon l'installation choisie).

## Implémentation du token

- `contracts/IERC20.sol` — interface standard EIP-20 (fournie, ne pas modifier)
- `contracts/MyToken.sol` — implémentation complète : erreurs custom, `onlyOwner`, `transfer`/`approve`/`transferFrom`, `mint`/`burn`/`transferOwnership`, pattern CEI et `unchecked` justifié dans `_transfer`/`_mint`/`_burn`

Compiler :

```bash
npx hardhat compile
forge build
```

## Tests Hardhat (TypeScript)

Fichier `test/MyToken.test.ts` — 11 tests couvrant déploiement, `transfer()`, `approve()`/`transferFrom()` (y compris allowance infinie), `mint()`/`burn()`.

```bash
# Lancer tous les tests avec rapport gas
npx hardhat test

# Avec couverture de code
npx hardhat coverage

# Régénérer les types TypeScript depuis les ABIs (après chaque compilation)
npx hardhat typechain
```

Le rapport de gas est généré dans `gas-report.txt`.

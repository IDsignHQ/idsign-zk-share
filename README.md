# zkVaults Module (DKMS)
ZK-MPC encrypted secret sharing module enabling the most private decentralized file sharing with no one in the middle, not event idSign; ONLY the wallets added to document can ever reconstruct and decrypt the documents shared via IPFS

---

### TL;DR

Imagine you want to send a secret message or file to someone or to a group of members such as a DAO. With zkVaults, you can encrypt the data on the frontend with a unique generated key and this key gets stored on 3 nodes on an MPC blockchain (partisiablockchain.com) or any Advanced MPC blockchain. The other members can then request the secret key, and decrypt the data that was stored publically but encrypted.

---

### **Objective**: 
Securely share a secret among a predefined set of users/wallets, authorized via their wallet addresses, using zero-knowledge proofs to maintain privacy.

### **Components**:

- **Smart Contract or Decentralized Storage:** For Storing The resourceID, encryptedSecret, and its ACLs (Access Control Lists). These values can be stored publicly leveraging the “no single point of failure” nature of this technology but at the same time ensuring privacy of the stored secret limiting it to a set of authorized users
- **zkVaults module:** For generating a new share, granting access of a share and modifying ACLs of an existing share. This should be implemented through programmable MPC actions leveraging computational rules done by multiple on-chain nodes (zkSmartContract)

### **Use Cases:**

- End-to-end Encrypted on-chain messaging
- Decentralized file-sharing (sharing the decryption key of an encrypted file stored on IPFS)
- Sharing information privately between DAO members
- Token-Gating without relying on a server to “Gate” the resource or the key to the resource
- Many more…

## How to deploy

Having run the example you may wish to try to deploy these contracts yourself. 

If you have made any modifications to the code, make sure that you understand how data is moved from
PBC to Ethereum to ensure that it can still happen in a secure manner.

If you wish to know more, you can read details on how it works [here](https://partisiablockchain.gitlab.io/documentation/smart-contracts/pbc-as-second-layer/how-to-create-your-own-second-layer-solution.html).

More information on how to deploy the solution can be found [here](https://partisiablockchain.gitlab.io/documentation/smart-contracts/pbc-as-second-layer/how-to-deploy-your-second-layer-solution.html).

### Prerequisites

To be able to compile the PBC smart contract you must have the 
[partisia-contract compiler](https://crates.io/crates/cargo-partisia-contract) installed. 
See the [documentation](https://partisiablockchain.gitlab.io/documentation/smart-contracts/install-the-smart-contract-compiler.html)
for a guide on how to install the compiler.

To compile and deploy the solidity contract you must have 
[node.js](https://nodejs.org/en) installed.

To make the deployment configuration available for the deployment scripts, create a `.env` file in 
the `./` directory, using the following template.

```text

```

### Build and deploy PBC private smart contract

From working directory `./`.

1. Build the PBC contract using the command 
    ```shell
    cargo partisia-contract build --release
    ```
    The outputs are located at `./target/wasm32-unknown-unknown/release/zk_vaults.{abi, zkwa}`.
2. [Deploy PBC contract](https://partisiablockchain.gitlab.io/documentation/smart-contracts/compile-and-deploy-contracts.html) on 
   [PBC testnet](https://browser.testnet.partisiablockchain.com/contracts/deploy). 
3. Copy the address of the deployed contract and paste it into the `PBC_CONTRACT_ADDRESS` field in 
   the `.env` file.
4. Copy each of the public keys of the ZK nodes in order and paste them into the 
   `ZK_NODE_PUBLIC_KEY_1`, `ZK_NODE_PUBLIC_KEY_2`, `ZK_NODE_PUBLIC_KEY_3` and `ZK_NODE_PUBLIC_KEY_4`
   fields in the `.env` file in order.

### Deploy solidity contract


1. Fill in the `GOERLI_API_URL` and `GOERLI_PRIVATE_KEY` fields in the `.env` file.
2. In the `hardhat.config.js` file, uncomment the lines
   ```text
   defaultNetwork: "goerli",
   ```
   and 
   ```text
   goerli: {
     url: GOERLI_API_URL,
     accounts: [`0x${GOERLI_PRIVATE_KEY}`],
   },
   ```
   to enable the Goerli network when running hardhat scripts.
3. Run the commands
   ```shell
   npm install
   npx hardhat run scripts/deploy.js --network goerli
   ```
   
Verifying the contract on [goerli.etherscan.io](https://goerli.etherscan.io/) makes it possible to 
interact with the contract on goerli.etherscan.io.

1. Fill in the `GOERLI_CONTRACT_ADDRESS` and `ETHERSCAN_API_KEY` fields in the `.env` file.
2. Run the command
   ```shell
   npx hardhat run scripts/verify.js --network goerli
   ```

## Helper scripts

The `./scripts` directory contains several scripts to help with deploying the solidity
contract to Ethereum.

### deploy.js

This script help with deploying the lib.sol contract, using the hardhat runtime.

To work correctly, it expects that the environment variables `GOERLI_API_URL`, `GOERLI_PRIVATE_KEY`, 
`PBC_CONTRACT_ADDRESS`, `ZK_NODE_PUBLIC_KEY_1`, `ZK_NODE_PUBLIC_KEY_2`, `ZK_NODE_PUBLIC_KEY_3`, and 
`ZK_NODE_PUBLIC_KEY_4` are set. See above for details.

Given the environment variables, the script formats them to form valid constructor arguments before
deploying.

The deploy.js script can be run with the command:

```shell
npx hardhat run scripts/deploy.js --network goerli
```

### verify.js

This script can be used to verify a deployed contract on the 
[goerli.etherscan.io](https://goerli.etherscan.io/) block explorer.

Verifying contract provides etherscan with an ABI for the contract, making it possible to display
the contract state and to interact with the contract via the etherscan UI.

For the script to work it expects the same environment variables as the deploy.js script.
Additionally, the variables `GOERLI_CONTRACT_ADDRESS` and `ETHERSCAN_API_KEY` must also be available.

The script formats the input from the deploy.js script, which is needed when verifying.

The script can be run with the command

```shell
npx hardhat run scripts/verify.js --network goerli
```

### pbc.js

This script can convert a public key, provided by a computation node on PBC, to an Ethereum address.

It is used by the deploy.js and verify.js scripts to format the constructor arguments.
It can also be used from the command line, to manually verify that the addresses on Ethereum 
are correctly derived from a public key from PBC.

Given a public key from PBC `A/J83e6pRe9ARxCJNrUxu2iVfh3HTKk4CEYyZFxWn4NG`, the script can be run
with the command

```shell
npx run-func scripts/pbc.js testKey "A9SqNrfygSuXOLNsdy4Gx8d0kSV5S/ET7GCnTVz90FQ7"
```

```text
Deriving from:    A9SqNrfygSuXOLNsdy4Gx8d0kSV5S/ET7GCnTVz90FQ7
EC point:         d4aa36b7f2812b9738b36c772e06c7c7749125794bf113ec60a74d5cfdd0543bab6ac00abfa468e83e14ea45541e7d3c479c9af4b8d7a83ce8300a88fc20043d
Ethereum Address: b6e8E2BD838518F6Ac146fa6f7271c023723C0d0
```


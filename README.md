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
- **zkVaults module:** For generating a new vault, granting access of a vault to users in its ACLs. This should be implemented through programmable MPC actions leveraging computational rules done by multiple on-chain nodes (zkSmartContract)

### **Use Cases:**

- End-to-end Encrypted on-chain messaging
- Decentralized file-sharing (sharing the decryption key of an encrypted file stored on IPFS)
- Sharing information privately between DAO members
- Token-Gating without relying on a server to “Gate” the resource or the key to the resource
- Many more…

## Documentation

### Key Functions

- Initialization (initialize):
   - Sets up the contract with the creator as the owner.
   - Initializes an empty list of vaults.

- Creating a Vault (create_vault):
   - A user (e.g., user1) can create a vault by providing a secret key, their address (owner), and a list of addresses that can access the vault (access control list or ACL).
   - The function returns an index for the newly created vault (vault_index) and an index for the secret key (secret_index).

- Reading a Vault (read_vault):
   - Another user (e.g., user2) can read the vault's secret key by providing the vault_index, secret_index, and a fresh RSA public key.
   - The function checks if the user is authorized to access the vault.
   - If authorized, it retrieves the secret key, encrypts it with the provided RSA public key, and returns the encrypted key.
   - The encrypted key can then be decrypted on the frontend using the corresponding RSA private key, ensuring the secret remains safe during transmission.

### Mock Frontent Sample Interaction 
```js
import init, { initialize, create_vault, read_vault } from 'zk_vaults';
import NodeRSA from 'node-rsa';

(async () => {
  // Initialize the Wasm module
  await init();

  // Generate RSA keys
  const key = new NodeRSA({ b: 2048 });
  const publicKey = key.exportKey('public');
  const privateKey = key.exportKey('private');

  console.log('Public Key:', publicKey);
  console.log('Private Key:', privateKey);

  // Contract context and addresses (replace with actual values)
  const context = { sender: '0xSenderAddress' };
  const owner = '0xOwnerAddress';
  const acls = ['0xAclAddress1', '0xAclAddress2'];
  const secretKey = 123456789n;

  // Initialize the contract
  const contractState = initialize(context, {});

  // Create a vault
  const { state, def } = create_vault(context, contractState, {}, secretKey, owner, acls);
  const vaultIndex = state.vaults.length - 1;
  const secretIndex = def.metadata;

  console.log('Vault Index:', vaultIndex);
  console.log('Secret Index:', secretIndex);

  // Read the vault
  const encryptedData = read_vault(context, state, {}, vaultIndex, secretIndex, publicKey);

  console.log('Encrypted Data:', encryptedData);

  // Decrypt the message
  const decryptedMessage = key.decrypt(encryptedData, 'utf8');

  console.log('Decrypted Message:', decryptedMessage);
})();
```

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

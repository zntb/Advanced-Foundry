# Transaction Types Introduction

To gain a clearer understanding of transaction types, let's return to Remix and deploy again a simple smart contract, such as `SimpleStorage.sol`.

We'll utilize the ZK Sync Remix plugin for deployment, ensuring our environment is set to 'wallet' and connected via MetaMask. After compiling the contract, we proceed to deploy it. But instead of sending a transaction directly, MetaMask will prompt us to **sign a message** 🖊️.

This MetaMask **signature request** displays details about the message, including transaction type, sender, recipient, and gas limit. This message shows an EIP712 message, with a transaction type `113`. Upon signing, the Remix terminal will then show the signed message's details like type, recipient, sender, gasLimit, gasPerPubdataByteLimit.

![MetaMask Signature Request](./assets/signature-request.png)

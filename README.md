# Test On zkSync

We can also run our `MerkleAirdrop.t::testUsersCanClaim` test on the zkSync chain.

To do this, we can start by switching to the zkSync version by running `foundryup --zksync`. Since the zkSync compiler operates differently from the standard solc compiler, it's better to verify that everything builds correctly before deploying.

```bash
forge build --ZK Sync
```

> 🗒️ **NOTE**:br
> If you encounter any warnings, they may be related to the use of `ecrecover`. These warnings can be safely ingnored since indicate that the accounts should use an ECDSA private key and should be EOAs. This warning are shown because the ZK Sync era supports native account abstraction.

Finally, we can run our tests on zkSync with the following command:

```bash
forge test --zksync -vvv
```

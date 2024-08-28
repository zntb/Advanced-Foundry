# Advanced EVM - Encoding

We can also accomplish our goals with the `bytecode` version directly. All you _really_ need to send a function call is the name of a function and the input types.

Two questions arise:

_**How do we send transactions that call functions with just the data field populated?**_

_**How do we populate the data field?**_

We're going to answer these by leveraging additional low-level keywords offered by Solidity, `staticcall` and `call`.

We've used call previously... if this code rings a bell:

```solidity
function withdraw(address recentWinner) public {
  (bool success, ) = recentWinner.call{ value: address(this).balance }("");
  require(success, "Transfer Failed");
}
```

**call:** How we call functions to change the state of the blockchain

**staticcall:** How we call view or pure functions

> ❗ **PROTIP** > `send` and `delegatecall` also exist as options for low-level calling to the blockchain, but we'll go over these in greater detail later!

When we write `recentWinner.call{value: address(this).balance}("");` we're directly updating the value property of the transaction we're sending. The parenthesis at the end of this call are where we provide our transaction data.

- within `{}` we're able to pass specific fields of a transaction, like `value`
- within `()` we can pass the data needed to call a specific function.

## Wrap Up

Whew, this is heavy, but it's advanced. The power provided by low-level function calls cannot be overstated.

Now that we have some understanding of how encoding can be using in sending transactions, let's take a step back in the next lesson to recap what we've gone over

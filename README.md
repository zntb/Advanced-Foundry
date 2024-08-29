# EVM signatures selectors

By passing this contract the address of our `CallAnything.sol` deployment. We're able to use the functions it possesses to interact with `CallAnything.sol`

Before we interact with anything, recall what the values of our storage variables on `CallAnything.sol` are currently.

Now we can call `callTransferFunctionDirectlyThree` on our `CallFunctionWithoutContract.sol` by passing a new address and amount. This should result in an updating of the storage variables on CallAnything.sol via this low-level call.

## Wrap Up

Hopefully by now you can see the power available through this methodology of low-level calls. Now, despite hyping it up for several lessons, low-level calls are risky, and it's worth noting that they should be avoided when possible. Use an interface or something similar if you can, because low-level calls can leave you open to a number of potential issues and vulnerabilities.

With that said, you've just learnt some really advanced stuff. If it's a little confusing, don't feel bad, you can always come back later when you've gained a little more experience and context of the EVM

If you're excited to learn more about how Solidity works under-the-hood, I recommend reading through the **[Deconstructing Solidity](https://blog.openzeppelin.com/deconstructing-a-solidity-contract-part-i-introduction-832efd2d7737)** series by OpenZeppelin. It does a great job breaking things down in a very digestible and granular way.

With that said, we're almost done, We've a couple things to tidy up in the section. Let's finish strong.

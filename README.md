# Create the depositAndMint function

Welcome back! I'm excited to keep going. So far our DSCEngine.sol has quite a bit of functionality already. We've the ability to mint DSC, we can deposit collateral, check account information and more.

The next thing I want to tackle is sort of the main function we'd expect in this protocol, depositCollateralAndMintDsc. This function should serve as a combination of the last two we wrote which allows users to deposit and mint in a single transaction.

The parameters for our depositCollateralAndMintDsc function are going to be similar to what we've seen in depositCollateral.

```solidity
function depositCollateralAndMintDsc(
  address tokenCollateralAddress,
  uint256 amountCollateral,
  uint256 amountDscToMint
) {}
```

All we really need to do, in this function, is call our depositCollateral and mintDsc functions in sequence.

> ❗ **NOTE**
> Both `depositCollateral` and `mintDsc` are current `external` functions. Set them to `public` before proceeding!

Because this is one of our main functions, we're absolutely going to add some NATSPEC.

```solidity
/*
 * @param tokenCollateralAddress: the address of the token to deposit as collateral
 * @param amountCollateral: The amount of collateral to deposit
 * @param amountDscToMint: The amount of DecentralizedStableCoin to mint
 * @notice: This function will deposit your collateral and mint DSC in one transaction
 */
function depositCollateralAndMintDsc(
  address tokenCollateralAddress,
  uint256 amountCollateral,
  uint256 amountDscToMint
) {
  depositCollateral(tokenCollateralAddress, amountCollateral);
  mintDsc(amountDscToMint);
}
```

## Wrap Up

This main function we've just written will allow users to deposit and mint in one transaction, greatly improving the UX of the protocol.

We've got great momentum, let's look at how users can redeem collateral, in the next lesson.

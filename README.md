# Create the redeem collateral function

## redeemCollateral

So far we've afforded our users a way to put money _into_ the protocol, they'll certainly need a way to get it out. Let's work through `redeemCollateral` next. This function is going to need to do a couple things:

1. Check that withdrawing the requested amount doesn't cause the account's `Health Factor` to break (fall below 1)
2. transfer the requested tokens from the protocol to the user

```solidity
function redeemCollateral(
  address tokenCollateralAddress,
  uint256 amountCollateral
) public moreThanZero(amountCollateral) nonReentrant {}
```

> ❗ **PROTIP**
> DRY: Don't Repeat Yourself. We'll be employing this concept from computer science later when we return to this function to refactor things.

We append the `moreThanZero` and `nonReentrant` modifiers to our function to prevent zero value transactions and as a safeguard for reentrancy.

With checks in place, we'll want to update the internal accounting of the contract to reflect the withdrawal. This updates contract state, so of course we'll want to emit a new event.

```solidity
function redeemCollateral(
  address tokenCollateralAddress,
  uint256 amountCollateral
) public moreThanZero(amountCollateral) nonReentrant {
  s_collateralDeposited[msg.sender][tokenCollateralAddress] -= amountCollateral;
  emit CollateralRedeemed(msg.sender, tokenCollateralAddress, amountCollateral);
}
```

> ❗**NOTE**
> We're relying on the Solidity compiler to revert if a user attempts to redeem an amount greater than their balance. More recent versions of the Solidity compiler protect against unsafe math.

Don't forget to add your event to the top of your contract as well.

```solidity
////////////////
//   Events   //
////////////////

event CollateralDeposited(
  address indexed user,
  address indexed token,
  uint256 indexed amount
);
event CollateralRedeemed(
  address indexed user,
  address indexed token,
  uint256 indexed amount
);
```

At this point in our function, we'll want to transfer the redeemed tokens to the user, but we're caught in a trap of sorts. Part of our requirements for this function is that the user's `Health Factor` mustn't be broken after the transfer as occured. In situations like these, you may see the `CEI (Checks, Effects, Interactions)` pattern broken sometimes. A protocol _could_ call a function prior to the transfer to calculate changes and determine if the `Health Factor` is broken, before a transfer occurs, but this is often quite gas intensive. For this reason protocols will often sacrifice `CEI` for efficiency.

```solidity
function redeemCollateral(
  address tokenCollateralAddress,
  uint256 amountCollateral
) public moreThanZero(amountCollateral) nonReentrant {
  s_collateralDeposited[msg.sender][tokenCollateralAddress] -= amountCollateral;
  emit CollateralRedeemed(msg.sender, tokenCollateralAddress, amountCollateral);

  bool success = IERC20(tokenCollateralAddress).transfer(
    msg.sender,
    amountCollateral
  );
  if (!success) {
    revert DSCEngine__TransferFailed();
  }

  _revertIfHealthFactorIsBroken(msg.sender);
}
```

This looks great. What does a user do when they want to exit the protocol entirely though? Redeeming all of their collateral through this function will revert due to the user's `Health Factor` breaking. The user would first need to burn their `DSC` to release their collateral. This two step process would be cumbersome (much liked `depositCollateral` and `mintDsc` was), so let's write the `burnDsc` function, then combine the two.

### burnDsc

In order for a user to burn their `DSC`, the tokens will need to be transferred to `address(0)`, and their balance within our `s_DSCMinted` mapping will need to be updated. Rather than transferring to `address(0)` ourselves, our function will take the tokens from the user and then call the inherent burn function on the token. We'll apply the `moreThanZero` modifier for our usual reasons (non-zero transactions only!).

```solidity
function burnDsc(uint256 amount) external moreThanZero(amount) {
  s_DSCMinted[msg.sender] -= amount;
  bool success = i_dsc.transferFrom(msg.sender, address(this), amount);
  if (!success) {
    revert DSCEngine__TransferFailed();
  }
}
```

The conditional above, should technically never hit, since transferFrom will revert with its own error if it fails, but we've a backstop, just in case.

We'll need to call burn on our `DSC` now.

```solidity
function burnDsc(uint256 amount) public moreThanZero(amount) {
  s_DSCMinted[msg.sender] -= amount;
  bool success = i_dsc.transferFrom(msg.sender, address(this), amount);
  if (!success) {
    revert DSCEngine__TransferFailed();
  }
  i_dsc.burn(amount);
  _revertIfHealthFactorIsBroken(msg.sender);
}
```

> ❗ **NOTE**
> We've added `_revertIfHealthFactorIsBroken`, but realistically, it should never hit, the user is burning "debt" and this should only improve the `Health Factor` of the account. A gas audit may remove this line.

### redeemCollateralForDsc

With both `redeemCollateral` and `burnDsc` written, we can now combine the functionality into one transaction.

```solidity
/*
 * @param tokenCollateralAddress: the collateral address to redeem
 * @param amountCollateral: amount of collateral to redeem
 * @param amountDscToBurn: amount of DSC to burn
 * This function burns DSC and redeems underlying collateral in one transaction
 */
function redeemCollateralForDsc(
  address tokenCollateralAddress,
  uint256 amountCollateral,
  uint256 amountDscToBurn
) external {
  burnDsc(amountDscToBurn);
  redeemCollateral(tokenCollateralAddress, amountCollateral);
}
```

### Wrap Up

Alright, the `DSCEngine` now has means for a user to both deposit and redeem collateral, as well as mint and burn `DSC`. We've also written functions which combine these calls to save steps in transactions.

As I mentioned briefly above, we're going to refactor these functions later on, but I wanted you to see the reasoning behind the refactor when we hit that point.

In the next lesson we approach liquidations! See you soon!

# Setup liquidations

## Liquidation

Our `DSCEngine` is almost done! We've got a couple more functions to flesh out still.

```solidity
function liquidate() external {}

function getHealthFactor() external view {}
```

We'll build out the `liquidate` function next, as it's an incredibly important pillar of how this protocol works.

Now that users are able to deposit collateral and mint, we need to protect against the protocol becoming `under-collateralized`. If the value of deposited collateral falls, such that users' `Health Factors` are broken, we need a method by which another user can `liquidate` those unhealthy positions to secure the value of the stablecoin.

Users who assist the protocol by liquidating unhealthy positions will be rewarded with the collateral for the position they've closed, which will exceed the value of the `DSC` burnt by virtue of our liquidation threshold.

To illustrate:

- User deposited $100 in collateral and mints $50 in `DSC`
- Collateral value falls to \$75, breaking the user's `Health Factor` (0.75)
- A `liquidator` burns \$50 in `DSC` to close the position
- The `liquidator` is rewarded \$75 in collateral
- The `liquidator` has profited \$25

Let's write this out.

```solidity
/*
* @param collateral: The ERC20 token address of the collateral you're using to make the protocol solvent again.
* This is collateral that you're going to take from the user who is insolvent.
* In return, you have to burn your DSC to pay off their debt, but you don't pay off your own.
* @param user: The user who is insolvent. They have to have a _healthFactor below MIN_HEALTH_FACTOR
* @param debtToCover: The amount of DSC you want to burn to cover the user's debt.
*
* @notice: You can partially liquidate a user.
* @notice: You will get a 10% LIQUIDATION_BONUS for taking the users funds.
* @notice: This function working assumes that the protocol will be roughly 150% overcollateralized in order for this
to work.
* @notice: A known bug would be if the protocol was only 100% collateralized, we wouldn't be able to liquidate
anyone.
* For example, if the price of the collateral plummeted before anyone could be liquidated.
*/
function liquidate(
  address collateral,
  address user,
  uint256 debtToCover
) external moreThanZero(debtToCover) nonReentrant {}
```

As such an important function to the protocol we've made every effort to be as clear and verbose in our `NATSPEC` as possible. If any of this isn't clear, or the motivations behind what we're doing seem confusing - use the resources you have to your advantage. Ask questions on **[Stack Exchange](https://ethereum.stackexchange.com/)** and **[Discord](https://discord.gg/cyfrin)**, open **[GitHub Discussions](https://github.com/Cyfrin/foundry-full-course-f23/discussions)**, use AI tools. Problem solving is paramount in software engineering.

Alright, let's keep going. The first thing we'll want to do in `liquidate` is verify that the passed user is eligible for liquidation. Someone being liquidated should have a `Health Factor` below `1`, otherwise this function should revert.

```solidity
function liquidate(
  address collateral,
  address user,
  uint256 debtToCover
) external moreThanZero(debtToCover) nonReentrant {
  uint256 startingUserHealthFactor = _healthFactor(user);
  if (startingUserHealthFactor > MIN_HEALTH_FACTOR) {
    revert DSCEngine__HealthFactorOk();
  }
}
```

Of course, we'll add our error to the errors section at the top of our contract.

```solidity
///////////////////
//     Errors    //
///////////////////

error DSCEngine__TokenAddressesAndPriceFeedAddressesAmountsDontMatch();
error DSCEngine__NeedsMoreThanZero();
error DSCEngine__TokenNotAllowed(address token);
error DSCEngine__TransferFailed();
error DSCEngine__BreaksHealthFactor(uint256 healthFactor);
error DSCEngine__MintFailed();
error DSCEngine__HealthFactorOk();
```

Our next step in the `liquidate` function is to remove the unhealthy position from the protocol, to do this we'll have to:

- burn the `DSC` debt being covered by the `liquidator` (not all of a position needs to be liquidated)
- calculate how much of the passes collateral type equates to the USD value of the debt being covered
- transfer the calculated amount of the passed collateral type to the `liquidator`
- updated internal accounting/balances

We'll need a new function to calculate this token amount, but we'll get to that next.

```solidity
function liquidate(
  address collateral,
  address user,
  uint256 debtToCover
) external moreThanZero(debtToCover) nonReentrant {
  uint256 startingUserHealthFactor = _healthFactor(user);
  if (startingUserHealthFactor > MIN_HEALTH_FACTOR) {
    revert DSCEngine__HealthFactorOk();
  }

  uint256 tokenAmountFromDebtCovered = getTokenAmountFromUsd(
    collateral,
    debtToCover
  );
}
```

We're passing the `getTokenAmountFromUsd` function the type of collateral, and the amount of debt we're covering. From this we'll be able to use price feeds to determine how much of the given collateral should be redeemed. This will be another public/external view function.

```solidity
//////////////////////////////////////////
//   Public & External View Functions   //
//////////////////////////////////////////

function getTokenAmountFromUsd(
  address token,
  uint256 usdAmountInWei
) public view returns (uint256) {
  AggregatorV3Interface priceFeed = AggregatorV3Interface(s_priceFeeds[token]);
  (, int256 price, , , ) = priceFeed.latestRoundData();

  return
    (usdAmountInWei * PRECISION) / (uint256(price) * ADDITIONAL_FEED_PRECISION);
}
```

Remember, we multiply by `PRECISION(1e18)` and `ADDITIONAL_FEED_PRECISION (1e10)` to assure our decimal precision is aligned in our numerator and denominator.

Next, let's ensure the `liquidator` is being incentivized for securing the protocol, we'll configure a `10%` bonus to the collateral awarded to the `liquidator`.

```solidity
function liquidate(
  address collateral,
  address user,
  uint256 debtToCover
) external moreThanZero(debtToCover) nonReentrant {
  uint256 startingUserHealthFactor = _healthFactor(user);
  if (startingUserHealthFactor > MIN_HEALTH_FACTOR) {
    revert DSCEngine__HealthFactorOk();
  }

  uint256 tokenAmountFromDebtCovered = getTokenAmountFromUsd(
    collateral,
    debtToCover
  );

  uint256 bonusCollateral = (tokeAmountFromDebtCovered * LIQUIDATION_BONUS) /
    LIQUIDATION_PRECISION;
}
```

Be sure to declare our new constant state variable, `LIQUIDATION_BONUS`. By setting this to `10` and dividing by our `LIQUIDATION_PRECISION` of `100`, we're setting a `10%` collateral bonus.

```solidity
/////////////////////////
//   State Variables   //
/////////////////////////

uint256 private constant LIQUIDATION_BONUS = 10;
```

We then of course add this bonus to our current `tokenAmountFromDebtCovered`, to acquire the total collateral being redeemed.

```solidity
function liquidate(
  address collateral,
  address user,
  uint256 debtToCover
) external moreThanZero(debtToCover) nonReentrant {
  uint256 startingUserHealthFactor = _healthFactor(user);
  if (startingUserHealthFactor > MIN_HEALTH_FACTOR) {
    revert DSCEngine__HealthFactorOk();
  }

  uint256 tokenAmountFromDebtCovered = getTokenAmountFromUsd(
    collateral,
    debtToCover
  );

  uint256 bonusCollateral = (tokeAmountFromDebtCovered * LIQUIDATION_BONUS) /
    LIQUIDATION_PRECISION;

  uint256 totalCollateralRedeemed = tokenAmountFromDebtCovered +
    bonusCollateral;
}
```

### Wrap Up

Whew, lots covered, and we're not quite done yet. In the next lesson we'll continue working on this `liquidate` function. We still need to burn the `liquidators` `DSC` and transfer the appropriate amount of collateral to them in exchange.

We'll also be doing a little bit of refactoring, so get ready and I'll see you there.

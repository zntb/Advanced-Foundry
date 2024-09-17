# Creating the deployment script

Once mocks are deployed, we can configure the anvilNetworkConfig with those deployed addresses, and return this struct.

```solidity
anvilNetworkConfig = NetworkConfig({
  wethUsdPriceFeed: address(ethUsdPriceFeed), // ETH / USD
  weth: address(wethMock),
  wbtcUsdPriceFeed: address(btcUsdPriceFeed),
  wbtc: address(wbtcMock),
  deployerKey: DEFAULT_ANVIL_PRIVATE_KEY,
});
```

Assure you add the `DEFAULT_ANVIL_PRIVATE_KEY` to our growing list of constant state variables.

```solidity
uint8 public constant DECIMALS = 8;
int256 public constant ETH_USD_PRICE = 2000e8;
int256 public constant BTC_USD_PRICE = 1000e8;
uint256 public constant DEFAULT_ANVIL_PRIVATE_KEY = 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80;
```

Great! With both of these functions written we can update our constructor to determine which function to call based on the block.chainid of our deployment.

```solidity
constructor() {
  if (block.chainid == 11155111) {
    activeNetworkConfig = getSepoliaEthConfig();
  } else {
    activeNetworkConfig = getOfCreateAnvilEthConfig();
  }
}
```

With the HelperConfig complete, we can return to DeployDSC.s.sol. Please reference the **[HelperConfig.s.sol within the GitHub repo](https://github.com/Cyfrin/foundry-defi-stablecoin-f23/blob/main/script/HelperConfig.s.sol)** if thing's haven't worked for you, or won't compile at this point.

## Back to DeployDSC

Returning to `DeployDSC.s.sol`, we can now import our HelperConfig and use it to acquire the the parameters for our deployments.

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.19;

import { Script } from "forge-std/Script.sol";
import { DecentralizedStableCoin } from "../src/DecentralizedStableCoin.sol";
import { DSCEngine } from "../src/DSCEngine.sol";
import { HelperConfig } from "./HelperConfig.s.sol";

contract DeployDSC is Script {
  function run() external returns (DecentralizedStableCoin, DSCEngine) {
    HelperConfig config = new HelperConfig();

    (
      address wethUsdPriceFeed,
      address wbtcUsdPriceFeed,
      address weth,
      address wbtc,
      uint256 deployerKey
    ) = config.activeNetworkConfig();
  }
}
```

With these values, we can now declare and assign our tokenAddresses and priceFeedAddresses arrays, and finally pass them to our deployments.

```solidity
...

address[] public tokenAddresses;
address[] public priceFeedAddresses;

function run() external returns (DecentralizedStableCoin, DSCEngine) {
    HelperConfig config = new HelperConfig();

    (address wethUsdPriceFeed, address wbtcUsdPriceFeed, address weth, address wbtc, uint256 deployerKey) = config.activeNetworkConfig();

    tokenAddresses = [weth, wbtc];
    priceFeedAddresses = [wethUsdPriceFeed, wbtcUsdPriceFeed];

    vm.startBroadcast();
    DecentralizedStableCoin dsc = new DecentralizedStableCoin();
    DSCEngine engine = new DSCEngine(tokenAddresses, priceFeedAddresses, address(dsc));
    vm.stopBroadcast();
}
```

Things look amazing so far, but there's one last thing we haven't really talked about. I'd mentioned in earlier lessons that we intend the DSCEngine to own and manage the DecentralizedStableCoin assets. DecentralizedStableCoin.sol is Ownable, and by deploying it this way, our msg.sender is going to be the owner by default. Fortunately, the Ownable library comes with the function `transferOwnership`. We'll just need to assure this is called in our deploy script.

```solidity
function run() external returns (DecentralizedStableCoin, DSCEngine) {
  HelperConfig config = new HelperConfig();

  (
    address wethUsdPriceFeed,
    address wbtcUsdPriceFeed,
    address weth,
    address wbtc,
    uint256 deployerKey
  ) = config.activeNetworkConfig();

  tokenAddresses = [weth, wbtc];
  priceFeedAddresses = [wethUsdPriceFeed, wbtcUsdPriceFeed];

  vm.startBroadcast();
  DecentralizedStableCoin dsc = new DecentralizedStableCoin();
  DSCEngine engine = new DSCEngine(
    tokenAddresses,
    priceFeedAddresses,
    address(dsc)
  );
  dsc.transferOwnership(address(engine));
  vm.stopBroadcast();
  return (dsc, engine);
}
```

### Wrap Up

Whew, not much left to say besides: Good work. In the next lesson, we'll be putting these scripts to the test with ... tests.

See you there!

**[DeployDSC.s.sol](https://github.com/Cyfrin/foundry-defi-stablecoin-f23/blob/main/script/DeployDSC.s.sol)**

**[HelperConfig.s.sol](https://github.com/Cyfrin/foundry-defi-stablecoin-f23/blob/main/script/HelperConfig.s.sol)**

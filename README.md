# Writing the deployment script

## Deploy Script

The muscle memory should be kicking in for some of these deploy scripts by now. Let's not waste any time setting this one up! Create a new file `script/DeployBasicNft.s.sol`.

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.18;

import { Script } from "forge-std/Script.sol";
import { BasicNft } from "../src/BasicNft.sol";

contract DeployBasicNft is Script {
  function run() external returns (BasicNft) {
    vm.startBroadcast();
    BasicNft basicNft = new BasicNft();
    vm.stopBroadcast();
    return basicNft;
  }
}
```

That's literally all there is to it. Run a quick `forge compile` as a sanity check to assure things build.

Great work!

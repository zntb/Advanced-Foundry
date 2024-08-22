# Deploy your ERC20 crypto currency

Now, by running `make anvil` (open a new terminal once your chain has started!) followed by `make deploy`...

## Wrap Up

Woo! Deployment to our anvil chain successful, let's go!

In the next lesson we'll test our contracts with the help of some AI tools and recap everything we've gone over so far. See you there!

## Test your ERC20 using AI

```solidity
function testTransfer() public {
  uint256 amount = 1000 * 10 ** 18; // Example amount
  vm.prank(msg.sender);
  ourToken.transfer(user1, amount);

  assertEq(ourToken.balanceOf(user1), amount);
  assertEq(ourToken.balanceOf(msg.sender), deployer.INITIAL_SUPPLY() - amount);
}

function testTransferFrom() public {
  uint256 amount = 500 * 10 ** 18; // Example amount
  vm.prank(msg.sender);
  ourToken.approve(user1, amount);

  vm.prank(user1);
  ourToken.transferFrom(msg.sender, user2, amount);

  assertEq(ourToken.balanceOf(user2), amount);
  assertEq(ourToken.allowance(msg.sender, user1), 0);
}

function testFailTransferExceedsBalance() public {
  uint256 amount = deployer.INITIAL_SUPPLY() + 1;
  vm.prank(msg.sender);
  ourToken.transfer(user1, amount); // This should fail
}

function testFailApproveExceedsBalance() public {
  uint256 amount = deployer.INITIAL_SUPPLY() + 1;
  vm.expectRevert();
  vm.prank(msg.sender);
  ourToken.approve(user1, amount); // This should fail
}
```

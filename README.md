# Create the deployment script

With these adjustments, our tests should function identically to before.

## Testing Flipping the URI

One thing we definitely haven't tested yet, and we should do quickly, is our flipMood function. Lets assure this properly swaps between happy and sad when called.

```solidity
function testFlipMoodIntegration() public {
  vm.prank(USER);
  moodNFT.mintNft();
  vm.prank(USER);
  moodNFT.flipMood(0);
  assert(
    keccak256(abi.encodePacked(moodNft.tokenURI(0))) ==
      keccak256(abi.encodePacked(SAD_SVG_URI))
  );
}
```

This test has our USER mint an NFT (which defaults as happy), and then flips the mood to sad with the flipMood function. We then assert that the token's URI matches what's expected.

Let's run it!

```shell
forge test --mt testFlipMoodIntegration
```

Uh oh. That ain't right.

### Wrap Up

Wow, this was a big lesson. We've written a deploy script and refactored some of our tests into more secure integration style tests.

For some reason `testFlipMoodIntegration` is erroring on us though...

In the next lesson we'll get some practice debugging, I suppose!

See you there!

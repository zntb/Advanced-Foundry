# Advanced EVM - Opcodes, calling, etc

The above may look like random numbers and letters to us, but to the `Ethereum Virtual Machine (EVM)`, this is effectively the alphabet it uses to perform computation. Every 2 bytes in the data above actually represents an op code. The website **[evm.codes](https://www.evm.codes/)** is an amazing resource for referencing these things.

You could almost use this reference like a dictionary. It tells us any time we see `00` in our bytecode, this represents the `STOP` operation, for example. In the bytecode example above, the first op code is `60`. This pertains to the PUSH1 operation!

This is what is meant by being `EVM Compatible`, `Polygon`, `Avalanche`, `Arbitrum` etc all compile to the same style of binary, readable by the `Ethereum Virtual Machine`.

Now, why are we talking about all this? How does it relate to abi encoding?

Until now we've only seen abi.encodePacked used to concatenate strings, but it's capable of much more.

## abi.encode

Strictly speaking, we can use abi encoding to encode anything we want into the bytecode format understood by the EVM.

Lets write a function to explore this. In our Encoding.sol file, in Remix add:

```solidity
function encodeNumber() public pure returns (bytes memory) {
  bytes memory number = abi.encode(1);
  return number;
}
```

> ❗ **IMPORTANT**
> ABI stands for application binary interface. What we've largely seen is the human readable version of an ABI.

Go ahead and compile/deploy Encoding.sol with this new function and call it. We should have the encoded version of the number `1` output.

This hex formmat, this encoding, is how a computer understands the number `1`.

Now, as mentioned, this can be used to encode basically anything, we can write a function to encode a string and see what it's output would be just the same.

```solidity
function encodeString() public pure returns (string memory) {
  byte memory someString = abi.encode("some string");
  return someString;
}
```

Something you may notice of each of our outputs is how many bytes of the output are comprised of zeros. This padding takes up a lot of space, whether or not it is important to the value being returned.

```bash
bytes: 0x0000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000b736f6d6520737472696e67000000000000000000000000000000000000000000
```

This is where **[abi.encodePacked](https://docs.soliditylang.org/en/latest/abi-spec.html#abi-packed-mode)** comes in and is available as a `non-standard packed mode`.

### abi.encodePacked

abi.encodePacked does much of the same encoding as abi.encode, but comes with some disclaimers.

- types shorter than 32 bytes are concatenated directly, without padding or sign extension

- dynamic types are encoded in-place and without the length.

- array elements are padded, but still encoded in-place

You can kind of think of encodePacked as a compressor which removed unnecessary padding of our binary objects.

We can demonstrate this in Remix by adding this function, redeploying our Encoding.sol contract and calling it.

```solidity
function encodeStringPacked() public pure returns (bytes memory) {
  bytes memory someString = abi.encodePacked("some string");
  return someString;
}
```

We can clearly see how much smaller the encodePacked output is, if we were trying to by gas efficient, the advantages of one over the other are obvious.

Encoding in this way is very similar to something else we've done before, typecasting. Add this function to Encoding.sol, and redeploy to see how these compare in practice:

```solidity
function encodeStringBytes() public pure returns (bytes memory) {
  bytes memory someString = bytes("some string");
  return someString;
}
```

So, it looks like abi.encodePacked and bytes casting are doing the same thing here, and for us - functionally they are - but behind the scenes things are a little more complicated. We won't go into the spefics here, but I encourage you to check out the deep dive in **[this forum post](https://forum.openzeppelin.com/t/difference-between-abi-encodepacked-string-and-bytes-string/11837)**.

### Decoding

Concatenating strings is fun and all, but in addition to _encoding_ things, we can also _decode_.

From the docs we can see the decode function takes the encoded data and a tuple of types to decode the data into.

```solidity
function decodeString() public pure returns (string memory) {
  string memory someString = abi.decode(encodeString(), (string));
  return someString;
}
```

Once again, we can add this function to our Encoding.sol contract and redeploy in remix to see how it works.

### Muli-Encoding/MultiDecoding

To take all this one step further, this encoding functionality affords us the ability to encode as much as we want. We can demonstrate this with the following functions:

```solidity
function multiEncode() public pure returns(bytes memory){
    bytes memory someString = abi.encode("some string", "it's bigger!");
    return someString;
}

function multiDecode() public pure returns(string memory, string memory){
    (string memory someString, string memory someOtherString) = abi.decode(multiEncode(),(string,string));
    return (someString, someOtherString)
}
```

When we multiEncode, you can see that our output is an _even bigger_ bytes object, with tonnes of padding. What do you think we can do about it?

You probably guessed, we can **also** multiEncodePacked. Try it out with:

```solidity
function multiEncodePacked() public pure returns (bytes memory) {
  bytes memory someString = abi.encodePacked("some string", "it's bigger!");
  return someString;
}
```

This is actually where our fun stops a little bit. Because we're packing the encoding of multiple strings, the decoding function is unable to properly split these up. It's not possible to multiDecode a multiEncodePacked object 😦. If you try something like:

```solidity
function multiDecodePacked()
  public
  pure
  returns (string memory, string memory)
{
  string memory someString = abi.decode(multiEncodePacked(), (string));
  return someString;
}
```

... this will actually error. We do have an alternative method though.

```solidity
function multiStringCastPacked() public pure returns (string memory) {
  string memory someString = string(multiEncodePacked());
  return someString;
}
```

This one actually _will_ work.

### Wrap Up

Don't feel back if this doesn't click right away, we're broaching some low-level concepts and functions here.

We're making great progress though and should have at least a somewhat better understanding of how the various methods of encoding and decoding are used in the EVM.

In the next lesson we'll apply these learnings and demonstrate how entire function calls can be encoded.

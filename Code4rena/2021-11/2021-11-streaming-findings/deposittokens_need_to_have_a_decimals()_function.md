## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [depositTokens need to have a decimals() function](https://github.com/code-423n4/2021-11-streaming-findings/issues/41) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
Only ERC20 tokens with a decimals() function can be used as a depositToken. A stream creator maybe not be aware of this restriction and the creation of a stream would revert.

## Proof of Concept
In the constructor of the Stream contract the decimals() (L310) functions of the depositToken is called. But according to EIP20 (https://eips.ethereum.org/EIPS/eip-20) the decimals() function is optional. 

https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L310

## Tools Used

## Recommended Mitigation Steps
- clearly inform the stream creator that the depositToken needs to have the decimals() function implemented


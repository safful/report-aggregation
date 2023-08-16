## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Permissions - return values not checked when sending ETH](https://github.com/code-423n4/2021-11-malt-findings/issues/329) 

# Handle

ScopeLift


# Vulnerability details

## Impact

On lines 85 and 101, ETH is transferred using a `.call` to an address provided as an input, but there is no verification that the call call succeeded. This can result in a call to `emergencyWithdrawGAS` or `partialWithdrawGAS` appearing successful but in reality it failed. This can happen when the provided `destination` address is a contract that cannot receive ETH, or if the `amount` provided is larger than the contract's balance

## Proof of Concept

Enter the following in remix, deploy the `Receiver` contract, and send 1 ETH when deploying the `Permissions` contract. Call `emergencyWithdrawGAS` with the receiver address and you'll see it reverts. This would not be caught in the current code

```solidity
pragma solidity ^0.8.0;

contract Receivier{}

contract Permissions {
    constructor() payable {}

    function emergencyWithdrawGAS(address payable destination) external {
        (bool ok, ) = destination.call{value: address(this).balance}('');
        require(ok, "call failed");
    }
}
```

## Tools Used

Remix

## Recommended Mitigation Steps

In `emergencyWithdrawGAS`:

```diff
- destination.call{value: address(this).balance}('');
+ (bool ok, ) = destination.call{value: address(this).balance}('');
+ require(ok, "call failed");
```

And similar for `partialWithdrawGAS`


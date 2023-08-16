## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [`assignDefaultFeatures` Does Nothing](https://github.com/code-423n4/2021-12-nftx-findings/issues/65) 

# Handle

leastwood


# Vulnerability details

## Impact

`assignDefaultFeatures` is intended to be called by the `dev` account, however, the function itself does not take in any arguments and instead sets the `enableRandomSwap` and `enableTargetSwap` state variables to itself.

## Proof of Concept

https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXVaultUpgradeable.sol#L111-L117
```
function assignDefaultFeatures() external {
    require(msg.sender == 0xDEA9196Dcdd2173D6E369c2AcC0faCc83fD9346a, "Not dev");
    enableRandomSwap = enableRandomRedeem;
    enableTargetSwap = enableTargetRedeem;
    emit EnableRandomSwapUpdated(enableRandomSwap);
    emit EnableTargetSwapUpdated(enableTargetSwap);
}
```

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider removing this function altogether or adding the necessary arguments such that the `dev` account can actually set the proper state variables.


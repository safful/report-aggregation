## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-02

# [Must approve 0 first](https://github.com/code-423n4/2022-12-tigris-findings/issues/104) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/main/contracts/Lock.sol#L117


# Vulnerability details

## Impact
Some tokens (like USDT) do not work when changing the allowance from an existing non-zero allowance value.They must first be approved by zero and then the actual allowance must be approved.


## Proof of Concept
https://github.com/code-423n4/2022-12-tigris/blob/main/contracts/Lock.sol#L117

```solidity
    function claimGovFees() public {
        address[] memory assets = bondNFT.getAssets();

        for (uint i=0; i < assets.length; i++) {
            uint balanceBefore = IERC20(assets[i]).balanceOf(address(this));
            IGovNFT(govNFT).claim(assets[i]);
            uint balanceAfter = IERC20(assets[i]).balanceOf(address(this));
            IERC20(assets[i]).approve(address(bondNFT), type(uint256).max);// @audit this could fail always with some tokens, 
            bondNFT.distribute(assets[i], balanceAfter - balanceBefore);
        }
    }
```

## Tools Used
manual revision

## Recommended Mitigation Steps



Add an approve(0) before approving;
```
    function claimGovFees() public {
        address[] memory assets = bondNFT.getAssets();

        for (uint i=0; i < assets.length; i++) {
            uint balanceBefore = IERC20(assets[i]).balanceOf(address(this));
            IGovNFT(govNFT).claim(assets[i]);
            uint balanceAfter = IERC20(assets[i]).balanceOf(address(this));
            IERC20(assets[i]).approve(address(bondNFT), 0);
            IERC20(assets[i]).approve(address(bondNFT), type(uint256).max);
            bondNFT.distribute(assets[i], balanceAfter - balanceBefore);
        }
  }
```
    

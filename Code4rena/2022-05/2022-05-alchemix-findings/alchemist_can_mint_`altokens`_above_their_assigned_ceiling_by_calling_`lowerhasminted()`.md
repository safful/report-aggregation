## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Alchemist can mint `AlTokens` above their assigned ceiling by calling `lowerHasMinted()`](https://github.com/code-423n4/2022-05-alchemix-findings/issues/166) 

# Lines of code

https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/AlchemicTokenV2Base.sol#L111-L124
https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/AlchemicTokenV2Base.sol#L189-L191


# Vulnerability details

## Impact
An alchemist / user can mint more than their alloted amount of AlTokens by calling `lowerHasMinted()` before they reach their minting cap.

## Proof of Concept
Function `mint()` in `AlchemicTokenV2Base.sol` 
```solidity
  function mint(address recipient, uint256 amount) external onlyWhitelisted {
    if (paused[msg.sender]) {
      revert IllegalState();
    }

    uint256 total = amount + totalMinted[msg.sender];
    if (total > mintCeiling[msg.sender]) {
      revert IllegalState();
    }

    totalMinted[msg.sender] = total;

    _mint(recipient, amount);
  }
```
Note the require conditional check that `total > mintCeiling[msg.sender]`.

In the same contract, there is the function `lowerHasMinted()` with the same permission level as mint and is thus callable by the same user as well.
```solidity
  function lowerHasMinted(uint256 amount) external onlyWhitelisted {
    totalMinted[msg.sender] = totalMinted[msg.sender] - amount;
  }
```

It is clear that a user can accumulate an infinite (within supply) amount of AlTokens by calling `lowerHasMinted()` before any action that would make them exceed their minting cap.

## Tools Used
Manual review, VScode

## Recommended Mitigation Steps
Change the permissioning on `lowerHasMinted()` to be restricted to a higher permissioned role like `onlySentinel()` , or deprecate this function as I could not find any uses of it throughout the codebase or in tests.


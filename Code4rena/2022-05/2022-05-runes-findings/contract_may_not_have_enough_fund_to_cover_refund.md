## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Contract may not have enough fund to cover refund](https://github.com/code-423n4/2022-05-runes-findings/issues/187) 

# Lines of code

https://github.com/code-423n4/2022-05-runes/blob/060b4f82b79c8308fe65674a39a07c44fa586cd3/contracts/ForgottenRunesWarriorsMinter.sol#L616-L619


# Vulnerability details

## Impact
Owner of the contract can call `withdrawAll` before the refund process is done to send all ETH to the vault. Since there are no payable receive function in `ForgottenRunesWarriorsMinter`, the owner won't be able to replenish the contract for the refund process.

## Proof of Concept
https://github.com/code-423n4/2022-05-runes/blob/060b4f82b79c8308fe65674a39a07c44fa586cd3/contracts/ForgottenRunesWarriorsMinter.sol#L616-L619

```solidity
    function withdrawAll() public payable onlyOwner {
        require(address(vault) != address(0), 'no vault');
        require(payable(vault).send(address(this).balance));
    }
```

## Recommended Mitigation Steps
Only allow owner to call `withdrawAll` after refund period


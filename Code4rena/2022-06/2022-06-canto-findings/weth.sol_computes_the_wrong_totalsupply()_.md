## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [WETH.sol computes the wrong totalSupply() ](https://github.com/code-423n4/2022-06-canto-findings/issues/191) 

# Lines of code

https://github.com/Plex-Engineer/lending-market/blob/ab31a612be354e252d72faead63d86b844172761/contracts/WETH.sol#L47


# Vulnerability details

## Impact

Affected code:

- [https://github.com/Plex-Engineer/lending-market/blob/ab31a612be354e252d72faead63d86b844172761/contracts/WETH.sol#L47](https://github.com/Plex-Engineer/lending-market/blob/ab31a612be354e252d72faead63d86b844172761/contracts/WETH.sol#L47)

`WETH.sol` is almost copied from the infamous WETH contract that lives in mainnet. This contract is supposed to receive the native currency of the blockchain (for example ETH) and wrap it into a tokenized, ERC-20 form. This contract computes the `totalSupply()` using the balance of the contract itself stored in the `balanceOf` mapping, when instead it should be using the native `balance` function. This way, `totalSupply()` always returns zero as the `WETH` contract itself has no way of calling `deposit` to itself and increase its own balance

## Proof of Concept

1. Alice transfers 100 ETH to `WETH.sol`
2. Alice calls `balanceOf()` for her address and it returns 100 WETH
3. Alice calls `totalSupply()`, expecting to see 100 WETH, but it returns 0

## Tools Used

Editor

## Recommended Mitigation Steps

```jsx
function totalSupply() public view returns (uint) {
    return address(this).balance
}
```


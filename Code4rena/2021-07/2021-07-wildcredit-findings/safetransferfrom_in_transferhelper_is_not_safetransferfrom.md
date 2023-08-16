## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [safeTransferFrom in TransferHelper is not safeTransferFrom](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/67) 

# Handle

jonah1005


# Vulnerability details

## Impact
A non standard erc20 token would always raise error when calling `_safeTransferFrom`.  If a user creates a USDT/DAI pool and deposit into the pool he would find out there's never a counterpart deposit.

## Proof of Concept

https://github.com/code-423n4/2021-07-wildcredit/blob/82c48d73fd27a9d4d5d4a395b3affcef4ef6c5c8/contracts/TransferHelper.sol#L19

TransferHelper does not uses `SafeERC20` library as the function name implies. 

A sample POC:
script:
```
usdt.functions.approve(lending_pair.address, deposit_amount).transact({'from': w3.eth.accounts[0]})
lending_pair.functions.deposit(w3.eth.accounts[0], usdt.address, deposit_amount).transact({'from': w3.eth.accounts[0]})
```

Error Message:
```
  Error: Transaction reverted: function returned an unexpected amount of data
      at LendingPair._safeTransferFrom (contracts/TransferHelper.sol:20)
      at LendingPair.deposit (contracts/LendingPair.sol:95)
```
## Tools Used

Hardhat

## Recommended Mitigation Steps
Uses openzeppelin `SafeERC20` in transfer helper (and any other contract that uses IERC20).


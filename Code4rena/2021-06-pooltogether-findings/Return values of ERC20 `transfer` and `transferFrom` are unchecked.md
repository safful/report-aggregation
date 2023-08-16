## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- SushiYieldSource
- BadgerYieldSource

# [Return values of ERC20 `transfer` and `transferFrom` are unchecked](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/112) 

# Handle

shw


# Vulnerability details

## Impact

In the contracts `BadgerYieldSource` and `SushiYieldSource`, the return values of ERC20 `transfer` and `transferFrom` are not checked to be `true`, which could be `false` if the transferred tokens are not ERC20-compliant (e.g., `BADGER`). In that case, the transfer fails without being noticed by the calling contract.

## Proof of Concept

If warden's understanding of the `BadgerYieldSource` is correct, the `badger` variable should be the `BADGER` token at address `0x3472a5a71965499acd81997a54bba8d852c6e53d`. However, this implementation of `BADGER` is not ERC20-compliant, which returns `false` when the sender does not have enough token to transfer (both for `transfer` and `transferFrom`). See the [source code on Etherscan](https://etherscan.io/address/0x3472a5a71965499acd81997a54bba8d852c6e53d#code) (at line 226) for more details.

Referenced code:
[BadgerYieldSource.sol#L44](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/BadgerYieldSource.sol#L44)
[BadgerYieldSource.sol#L79](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/BadgerYieldSource.sol#L79)
[SushiYieldSource.sol#L48](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/SushiYieldSource.sol#L48)
[SushiYieldSource.sol#L89](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/SushiYieldSource.sol#L89)

## Recommended Mitigation Steps

Use the `SafeERC20` library [implementation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol) from Openzeppelin and call `safeTransfer` or `safeTransferFrom` when transferring ERC20 tokens.


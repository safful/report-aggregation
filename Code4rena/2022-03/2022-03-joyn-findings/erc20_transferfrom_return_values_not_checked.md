## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [ERC20 transferFrom return values not checked](https://github.com/code-423n4/2022-03-joyn-findings/issues/52) 

# Lines of code

https://github.com/code-423n4/2022-03-joyn/blob/main/core-contracts/contracts/ERC721Payable.sol#L54


# Vulnerability details

## Details

The `transferFrom()` function returns a boolean value indicating success. This parameter needs to be checked to see if the transfer has been successful. Oddly, `transfer()` function calls were checked.

Some tokens like [EURS](https://etherscan.io/address/0xdb25f211ab05b1c97d595516f45794528a807ad8#code) and [BAT](https://etherscan.io/address/0x0d8775f648430679a709e98d2b0cb6250d2887ef#code) will **not** revert if the transfer failed but return `false` instead. Tokens that don't actually perform the transfer and return `false` are still counted as a correct transfer.

## Impact

Users would be able to mint NFTs for free regardless of mint fee if tokens that don’t revert on failed transfers were used.

## Recommended Mitigation Steps

Check the `success` boolean of all `transferFrom()` calls. Alternatively, use OZ’s `SafeERC20`’s `safeTransferFrom()` function.


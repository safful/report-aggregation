## Tags

- bug
- disagree with severity
- QA (Quality Assurance)
- sponsor acknowledged
- sponsor confirmed

# [`Safe.sol` allow success transaction to fail](https://github.com/code-423n4/2022-06-illuminate-findings/issues/100) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Safe.sol#L82-L105
https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/marketplace/Safe.sol#L82-L105
https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/redeemer/Safe.sol#L82-L105


# Vulnerability details


## `Safe.sol` allow success transaction to fail
Some ERC20 token implementations return more than 32 bytes(common Curve token). This can be a real concern since all core component using the same Safe Transfer Lib 
 After some test using  [https://github.com/Rari-Capital/solmate.git]

```
[BAIL] testTransferFromWithReturnsTooMuch()
[FAIL] testTransferFromWithReturnsTooMuch(address,address,uint256,bytes). Counterexample: (0x00000000000000000000000000000000000ffffE, 0x0000000000000000000000000000000000000001, 115792089237316195423570985008687907853269984665640564039457584007913129639935, 0x)
[BAIL] testApproveWithReturnsTooMuch()
[FAIL] testFailApproveWithReturnsTwo(address,uint256,bytes). Counterexample: (0x000000000000000000000000000000000004C0E4, 0, 0x)
[FAIL] testApproveWithReturnsTooMuch(address,uint256,bytes). Counterexample: (0x00000000000000000000000000000000000ffffE, 1, 0x)
[FAIL] testTransferWithReturnsTooMuch(address,uint256,bytes). Counterexample: (0x00000000000000000000000000000000000fFFFf, 27379694730619439495811032571422462501613862458272780721729846947764021554765, 0x)
[BAIL] testTransferWithReturnsTooMuch()
[FAIL] testFailTransferWithGarbage(address,uint256,bytes,bytes). Counterexample: (0x00000000000000000000000000000000000F40ae, 68959440145808540021340254471931488759807906017241826129032747446, 0x15dedf2835614b24353f20baa93783e58366c16defd811eddd5de9d9057489ca, 0xd946b45606da60c706edf8c4eb76e0d281d8f063f14850d952ea10c697f0931756bec369411fa90953093f29086d44a50b2cf829a5815f5a72124ff447ac4a69)
``` 

## Impact
If a transaction fail or success will not be considered and this will pass any call of Safe lib

## Proof of Concept 
https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Safe.sol

Safe lib used here
```
./lender/Lender.sol
./marketplace/marketplace.sol
./redeemer/redeemer.sol
```

    1- no data
    if iszero(r) {
        // Copy the revert message into memory.
        returndatacopy(0, 0, returnDataSize)

        // Revert with the same message.
        revert(0, returnDataSize)
    }

    2- returndatasize > 32 bytes
    default {
        // It returned some malformed input.
        result := 0
    }

    3- reverting
    require(success(result), 'transfer from failed');

### Tools Used
dapp-tools, vim

### Recommended Mitigation Steps
Use the latest Safe Transfer lib available



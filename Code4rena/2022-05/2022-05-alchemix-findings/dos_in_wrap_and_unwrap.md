## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [DoS in wrap and unwrap](https://github.com/code-423n4/2022-05-alchemix-findings/issues/159) 

# Lines of code

https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/adapters/fuse/FuseTokenAdapterV1.sol#L76
https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/adapters/fuse/FuseTokenAdapterV1.sol#L98


# Vulnerability details

## Impact
the code is doing wrong check, so when things will work it will revert.


## Proof of Concept
In the function `wrap()` there is this lines:
```
   if ((error = ICERC20(token).mint(amount)) != NO_ERROR) {
            revert FuseError(error);
        }
```
but `mint` returns the amount that minted, so when `error = amount` the check will fail even though it worked good.

Same in `unwrap`:
```
if ((error = ICERC20(token).redeem(amount)) != NO_ERROR) {
            revert FuseError(error);
        }
```
the redeem returns the amount.


## Recommended Mitigation Steps

I recommend to change the lines like this:
    in wrap:
    ```
     if ((error = ICERC20(token).mint(amount)) != amount) {
            revert FuseError(error);
        }
    ```
    and in unwrap:
    ```
    if ((error = ICERC20(token).redeem(amount)) != amount) {
            revert FuseError(error);
        }
    ```



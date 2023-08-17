## Tags

- bug
- disagree with severity
- QA (Quality Assurance)
- sponsor confirmed

# [lockFor() in BkdLocker don't check that user is not 0x0 and if user by mistake call this function with value 0x0 s/he is going to lose his funds.](https://github.com/code-423n4/2022-05-backd-findings/issues/166) 

# Lines of code

https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/BkdLocker.sol#L227-L232


# Vulnerability details

## Impact
function `lockFor()`  in `BkdLocker` is supposed to lock 'msg.sender` funds and increase `user` address funds but if anyone one calls it with `0x0` address by mistake then his funds will be locked forever.

## Proof of Concept
This is `lockFor()` code in `BkdLocker`:
```
    function lockFor(address user, uint256 amount) public override {
        govToken.safeTransferFrom(msg.sender, address(this), amount);
        _userCheckpoint(user, amount, balances[user] + amount);
        totalLocked += amount;
        emit Locked(user, amount);
    }
```
As you can see there is no check that `user` is not `0x0`. code calls `_userCheckpoint()` which will increase `0x0` balances in the contract and there is no check in `_userCheckpoint()` either and user can lose all his funds just by one simple mistake.

## Tools Used
VIM

## Recommended Mitigation Steps
check that `user` is not `0x0` in `lcokFor`


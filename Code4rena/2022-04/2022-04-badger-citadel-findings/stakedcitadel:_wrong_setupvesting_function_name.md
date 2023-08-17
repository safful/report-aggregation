## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [StakedCitadel: wrong setupVesting function name](https://github.com/code-423n4/2022-04-badger-citadel-findings/issues/9) 

# Lines of code

https://github.com/code-423n4/2022-04-badger-citadel/blob/main/src/StakedCitadel.sol#L830


# Vulnerability details

## Impact
In the _withdraw function of the StakedCitadel contract, the setupVesting function of vesting is called, while in the StakedCitadelVester contract, the function name is vest, which will cause the _withdraw function to fail, so that the user cannot withdraw the tokens.

```
        IVesting(vesting).setupVesting(msg.sender, _amount, block.timestamp);
        token.safeTransfer(vesting, _amount);
        ...
    function vest(
        address recipient,
        uint256 _amount,
        uint256 _unlockBegin
    ) external {
        require(msg.sender == vault, "StakedCitadelVester: only xCTDL vault");
        require(_amount > 0, "StakedCitadelVester: cannot vest 0");

        vesting[recipient].lockedAmounts =
            vesting[recipient].lockedAmounts +
            _amount;
        vesting[recipient].unlockBegin = _unlockBegin;
        vesting[recipient].unlockEnd = _unlockBegin + vestingDuration;

        emit Vest(
            recipient,
            vesting[recipient].lockedAmounts,
            _unlockBegin,
            vesting[recipient].unlockEnd
        );
    }
```
## Proof of Concept
https://github.com/code-423n4/2022-04-badger-citadel/blob/main/src/StakedCitadel.sol#L830
https://github.com/code-423n4/2022-04-badger-citadel/blob/main/src/interfaces/citadel/IVesting.sol#L5

## Tools Used
None
## Recommended Mitigation Steps

Use the correct function name
```
interface IVesting {
    function vest(
        address recipient,
        uint256 _amount,
        uint256 _unlockBegin
    ) external;
}
...
IVesting(vesting).vest(msg.sender, _amount, block.timestamp);
token.safeTransfer(vesting, _amount);
```


## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [The fund function of the CDSTemplate contract does not match the description](https://github.com/code-423n4/2022-01-insure-findings/issues/161) 

# Handle

cccz


# Vulnerability details

## Impact

The fund function of the CDSTemplate contract does not match the description, the caller will not receive any iToken after sending tokens, and the owner can take away the tokens in surplusPool.

```
    /**
     * @notice A liquidity provider supplies collatral to the pool and receives iTokens
     * @param _amount amount of token to deposit
     */
    function fund(uint256 _amount) external {
        require(paused == false, "ERROR: PAUSED");

        //deposit and pay fees
        uint256 _attribution = vault.addValue(
            _amount,
            msg.sender,
            address(this)
        );

        surplusPool += _attribution;

        emit Fund(msg.sender, _amount, _attribution);
    }

    function defund(uint256 _amount) external override onlyOwner {
        require(paused == false, "ERROR: PAUSED");

        uint256 _attribution = vault.withdrawValue(_amount, msg.sender);
        surplusPool -= _attribution;

        emit Defund(msg.sender, _amount, _attribution);
    }
```

## Proof of Concept

https://github.com/code-423n4/2022-01-insure/blob/main/contracts/CDSTemplate.sol#L156-L182

## Tools Used

Manual analysis

## Recommended Mitigation Steps

Change the description of the fund function or send iToken to the caller


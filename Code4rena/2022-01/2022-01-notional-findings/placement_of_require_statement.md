## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Placement of require statement](https://github.com/code-423n4/2022-01-notional-findings/issues/55) 

# Handle

Jujic


# Vulnerability details

## Impact
The require statement can be placed earlier (`before get coolDown`) to reduce gas usage. 

## Proof of Concept
https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/sNOTE.sol#L240
```
function redeem(uint256 sNOTEAmount) external nonReentrant {
        AccountCoolDown memory coolDown = accountCoolDown[msg.sender];
        require(sNOTEAmount <= balanceOf(msg.sender), "Insufficient balance");
        require(
            coolDown.redeemWindowBegin != 0 &&
            coolDown.redeemWindowBegin < block.timestamp &&
            block.timestamp < coolDown.redeemWindowEnd,
            "Not in Redemption Window"
        );

        uint256 bptToRedeem = getPoolTokenShare(sNOTEAmount);
        _burn(msg.sender, bptToRedeem);

        BALANCER_POOL_TOKEN.safeTransfer(msg.sender, bptToRedeem);
    }
```
## Tools Used
Remix
## Recommended Mitigation Steps
Relocate the require statement upper.


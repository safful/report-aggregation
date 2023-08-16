## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [[WP-M17] `USDMPegRecovery.sol#withdraw()` withdraw may often fail](https://github.com/code-423n4/2022-02-concur-findings/issues/212) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/USDMPegRecovery.sol#L110-L128


# Vulnerability details

Per the doc:

> USDM deposits are locked based on the KPI’s from carrot.eth.

> 3Crv deposits are not locked.

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/USDMPegRecovery.sol#L110-L128

```solidity
function withdraw(Liquidity calldata _withdrawal) external {
        Liquidity memory total = totalLiquidity;
        Liquidity memory user = userLiquidity[msg.sender];
        if(_withdrawal.usdm > 0) {
            require(unlockable, "!unlock usdm");
            usdm.safeTransfer(msg.sender, uint256(_withdrawal.usdm));
            total.usdm -= _withdrawal.usdm;
            user.usdm -= _withdrawal.usdm;
        }

        if(_withdrawal.pool3 > 0) {
            pool3.safeTransfer(msg.sender, uint256(_withdrawal.pool3));
            total.pool3 -= _withdrawal.pool3;
            user.pool3 -= _withdrawal.pool3;
        }
        totalLiquidity = total;
        userLiquidity[msg.sender] = user;
        emit Withdraw(msg.sender, _withdrawal);
    }
```

However, because the `withdraw()` function takes funds from the balance of the contract, once the majority of the funds are added to the curve pool via `provide()`. The `withdraw()` may often fail due to insufficient funds in the balance.

### PoC

1. Alice deposits `4M` USDM and `4M` pool3 tokens;
2. Guardian calls `provide()` and all the `usdm` and `pool3` to `usdm3crv`;
3. Alice calls `withdraw()`, the tx will fail, due to insufficient balance.

### Recommendation

Consider calling `usdm3crv.remove_liquidity_one_coin()`  when the balance is insufficient for the user's withdrawal.


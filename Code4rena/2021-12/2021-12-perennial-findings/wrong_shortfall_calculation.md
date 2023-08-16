## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [Wrong shortfall calculation](https://github.com/code-423n4/2021-12-perennial-findings/issues/18) 

# Handle

kenzo


# Vulnerability details

Every time an account is settled, if shortfall is created, due to a wrong calculation shortfall will double in size and add the new shortfall.

## Impact
Loss of funds: users won't be able to withdraw the correct amount of funds. Somebody would have to donate funds to resolve the wrong shortfall.

## Proof of Concept
We can see in the `settleAccount` of `OptimisticLedger` that self.shortfall ends up being self.shortfall+self.shortfall+newShortfall: [(Code ref)](https://github.com/code-423n4/2021-12-perennial/blob/main/protocol/contracts/collateral/types/OptimisticLedger.sol#L63:#L74)
```
    function settleAccount(OptimisticLedger storage self, address account, Fixed18 amount)
    internal returns (UFixed18 shortfall) {
        Fixed18 newBalance = Fixed18Lib.from(self.balances[account]).add(amount);

        if (newBalance.sign() == -1) {
            shortfall = self.shortfall.add(newBalance.abs());
            newBalance = Fixed18Lib.ZERO;
        }

        self.balances[account] = newBalance.abs();
        self.shortfall = self.shortfall.add(shortfall);
    }
```

Additionally, you can add the following line to the "shortfall reverts if depleted" test in Collateral.test.js, line 190:
```
await collateral.connect(productSigner).settleAccount(userB.address, -50)
```
Previously the test product had 50 shortfall. Now we added 50 more, but the test will print that the actual shortfall is 150, and not 100 as it should be.

## Recommended Mitigation Steps
Move the setting of `self.shortfall` to inside the if function and change the line to:
```
self.shortfall = shortfall
```


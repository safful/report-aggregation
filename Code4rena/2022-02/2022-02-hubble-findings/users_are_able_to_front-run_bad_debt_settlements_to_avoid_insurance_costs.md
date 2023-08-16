## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Users are able to front-run bad debt settlements to avoid insurance costs](https://github.com/code-423n4/2022-02-hubble-findings/issues/59) 

# Lines of code

https://github.com/code-423n4/2022-02-hubble/blob/main/contracts/InsuranceFund.sol#L71-L75
https://github.com/code-423n4/2022-02-hubble/blob/main/contracts/InsuranceFund.sol#L62-L69


# Vulnerability details

## Impact

A user is able to front-run the call to `seizeBadDebt()` in `InsuranceFund.sol` to avoid paying the insurance costs.

`seizeBadDebt()` is called by `MarginAccount.settleBadDebt()` which is a public function. When this functions is called the transaction will appear in the mem pool.  A user may then call `InsuranceFund.withdraw()` to withdraw all of their shares. If they do this with a higher gas fee it will likely be processed before the `settleBadDebt()` transaction. In this way they will avoid incurring any cost from the assets being seized.

The impact is that users may gain their share of the insurance funding payments with minimal risk (minimal as there is a change the front-run will not succeed) of having to repay these costs.

## Proof of Concept

```
    function withdraw(uint _shares) external {
        settlePendingObligation();
        require(pendingObligation == 0, "IF.withdraw.pending_obligations");
        uint amount = balance() * _shares / totalSupply();
        _burn(msg.sender, _shares);
        vusd.safeTransfer(msg.sender, amount);
        emit FundsWithdrawn(msg.sender, amount, block.timestamp);
    }
```

```
    function seizeBadDebt(uint amount) external onlyMarginAccount {
        pendingObligation += amount;
        emit BadDebtAccumulated(amount, block.timestamp);
        settlePendingObligation();
    }
```

## Recommended Mitigation Steps

Consider making the withdrawals a two step process. The first step requests a withdrawal and marks the time. The second request processes the withdrawal but requires a period of time to elapse since the first step.

To avoid having users constantly having pending withdrawal, each withdrawal should have an expiry time and also a recharge time. The if the second step is not called within expiry amount of time it should be considered invalid. The first step must not be able to be called until recharge time has passed.

Another solution involves a design change where the insurance fund is slowly filled up over time without external deposits. However, this has the disadvantage that bad debts received early in the protocols life time may not have sufficient insurance capital to cover them.


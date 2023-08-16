## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- resolved

# [Unchecked ERC20 transfers can cause lock up](https://github.com/code-423n4/2021-06-realitycards-findings/issues/2) 

# Handle

axic


# Vulnerability details

## Impact

Some major tokens went live before ERC20 was finalised, resulting in a discrepancy whether the transfer functions a) should return a boolean or b) revert/fail on error. The current best practice is that they should revert, but return “true” on success. However, not every token claiming ERC20-compatibility is doing this — some only return true/false; some revert, but do not return anything on success. This is a well known issue, heavily discussed since mid-2018.

Today many tools, including OpenZeppelin, offer a wrapper for “safe ERC20 transfer”: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol

RealityCards is not using such a wrapper, but instead tries to ensure successful transfers via the `balancedBooks` modifier:

```
    modifier balancedBooks {
        _;
        // using >= not == in case anyone sends tokens direct to contract
        require(
            erc20.balanceOf(address(this)) >=
                totalDeposits + marketBalance + totalMarketPots,
            "Books are unbalanced!"
        );
    }
```

This modifier is present on most functions, but is missing on `topupMarketBalance`:
```
    function topupMarketBalance(uint256 _amount) external override {
        erc20.transferFrom(msgSender(), address(this), _amount);
        if (_amount > marketBalanceDiscrepancy) {
            marketBalanceDiscrepancy = 0;
        } else {
            marketBalanceDiscrepancy -= _amount;
        }
        marketBalance += _amount;
    }
```

In the case an ERC20 token which is not reverting on failures is used, a malicious actor could call `topupMarketBalance` with a failing transfer, but also move the value of `marketBalance` above the actual holdings. After this, `deposit`, `withdrawDeposit`, `payRent`, `payout`, `sponsor`, etc. could be locked up and always failing with “Books are unbalanced”.

## Proof of Concept

Anyone can call `topupMarketBalance` with some unrealistically large number, so that `marketBalance` does not overflow, but is above the actually helping balances. This is only possible if the underlying ERC20 used is not reverting on failures, but return “false” instead.

## Tools Used

Manual review

## Recommended Mitigation Steps

1. Use something like OpenZeppelin’s SafeERC20
2. Set up an allow list for tokens, which are knowingly safe
3. Consider a different approach to the `balancedBooks` modifier



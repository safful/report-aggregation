## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Flashloan is given for 1 token but checks balances for both reward and deposit token](https://github.com/code-423n4/2021-11-streaming-findings/issues/50) 

# Handle

pedroais


# Vulnerability details

## Impact
Useless checks that cost gas
## Proof of Concept
Since the Flashloan function has the lock modifier reentrancy is not possible so checking both tokens is useless.

## Recommended Mitigation Steps


Proposed new function with less code :


            function flashloan(address token, address to, uint112 amount, bytes memory data) public lock {
        require(token == depositToken || token == rewardToken, "erc");

        uint256 preTokenBalance = ERC20(token).balanceOf(address(this));

        ERC20(token).safeTransfer(to, amount);

        // the `to` contract should have a public function with the signature:
        // function lockeCall(address initiator, address token, uint256 amount, bytes memory data);
        LockeCallee(to).lockeCall(msg.sender, token, amount, data);

        uint256 postTokenBalance = ERC20(token).balanceOf(address(this));

        uint112 feeAmt = amount * 10 / 10000; // 10bps fee
        require(preTokenBalance + feeAmt <= postTokenBalance, "f1");

        if (token == depositToken) {
            depositTokenFlashloanFeeAmount += feeAmt;
        } else {
            rewardTokenFeeAmount += feeAmt;
        }

        emit Flashloaned(token, msg.sender, amount, feeAmt);
    }



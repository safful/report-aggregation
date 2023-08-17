## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- old-submission-method

# [[Denial-of-Service] Contract Owner Could Block Users From Withdrawing Their Strike](https://github.com/code-423n4/2022-06-putty-findings/issues/296) 

# Lines of code

https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L500


# Vulnerability details

## Proof-of-Concept

When users withdraw their strike escrowed in Putty contract, Putty will charge a certain amount of fee from the strike amount. The fee will first be sent to the contract owner, and the remaining strike amount will then be sent to the users.

[https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L500](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L500)

```solidity
function withdraw(Order memory order) public {
	..SNIP..

	// transfer strike to owner if put is expired or call is exercised
	if ((order.isCall && isExercised) || (!order.isCall && !isExercised)) {
		// send the fee to the admin/DAO if fee is greater than 0%
		uint256 feeAmount = 0;
		if (fee > 0) {
			feeAmount = (order.strike * fee) / 1000;
			ERC20(order.baseAsset).safeTransfer(owner(), feeAmount);
		}

		ERC20(order.baseAsset).safeTransfer(msg.sender, order.strike - feeAmount);

		return;
	}
	..SNIP..
}
```

There are two methods on how the owner can deny user from withdrawing their strike amount from the contract

#### Method #1 - Set the `owner()` to `zero` address

Many of the token implementations do not allow transfer to `zero` address ([Reference](https://github.com/d-xo/weird-erc20#revert-on-transfer-to-the-zero-address)). Popular ERC20 implementations such as the following Openzeppelin's ERC20 implementation do not allow transfer to `zero` address, and will revert immediately if the `to` address (recipient) points to a `zero` address during a transfer.

[https://github.com/OpenZeppelin/openzeppelin-contracts/blob/5fbf494511fd522b931f7f92e2df87d671ea8b0b/contracts/token/ERC20/ERC20.sol#L226](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/5fbf494511fd522b931f7f92e2df87d671ea8b0b/contracts/token/ERC20/ERC20.sol#L226)

```solidity
function _transfer(
    address from,
    address to,
    uint256 amount
) internal virtual {
    require(from != address(0), "ERC20: transfer from the zero address");
    require(to != address(0), "ERC20: transfer to the zero address");

    _beforeTokenTransfer(from, to, amount);

    uint256 fromBalance = _balances[from];
    require(fromBalance >= amount, "ERC20: transfer amount exceeds balance");
    unchecked {
        _balances[from] = fromBalance - amount;
        // Overflow not possible: the sum of all balances is capped by totalSupply, and the sum is preserved by
        // decrementing then incrementing.
        _balances[to] += amount;
    }

    emit Transfer(from, to, amount);

    _afterTokenTransfer(from, to, amount);
}
```

It is possible for the owner to transfer the ownership to a `zero` address, thus causing the fee transfer to the contract owner to always revert. When the fee transfer always reverts, no one can withdraw their strike amount from the contract.

This issue will affect all orders that adopt a `baseAsset` that reverts when transferring to `zero` address.

#### Method #2 - If `baseAsset` is a ERC777 token

> Note: `owner()` could point to a contract or EOA account. By pointing to a contract, the contract could implement logic to revert whenever someone send tokens to it.

ERC777 contains a `tokensReceived` hook that will notify the recipient whenever someone sends some tokens to the recipient . 

Assuming that the `baseAsset` is a ERC77 token, the recipient, which is the `owner()` in this case, could always revert whenever `PuttyV2` contract attempts to send the fee to recipient. This will cause the `withdraw` function to revert too. As a result, no one can withdraw their strike amount from the contract.

This issue will affect all orders that has ERC777 token as its `baseAsset`.

## Impact

User cannot withdraw their strike amount and their asset will be stuck in the contract.

## Recommended Mitigation Steps

It is recommended to adopt a [withdrawal pattern](https://docs.soliditylang.org/en/v0.8.15/common-patterns.html#withdrawal-from-contracts) for retrieving owner fee.

Instead of transferring the fee directly to owner address during withdrawal, save the amount of fee that the owner is entitled to in a state variable. Then, implement a new function that allows the owner to withdraw the fee from the `PuttyV2` contract.

Consider the following implementation. In the following example, there is no way for the owner to perform denial-of-user because the outcome of the fee transfer (succeed or fail) to the owner will not affect the user's strike withdrawal process. 

This will give users more assurance and confidence about the security of their funds stored within Putty.

```solidity
mapping(address => uint256) public ownerFees;

function withdraw(Order memory order) public {
	..SNIP..
    // transfer strike to owner if put is expired or call is exercised
    if ((order.isCall && isExercised) || (!order.isCall && !isExercised)) {
        // send the fee to the admin/DAO if fee is greater than 0%
        uint256 feeAmount = 0;
        if (fee > 0) {
            feeAmount = (order.strike * fee) / 1000;
            ownerFees[order.baseAsset] += feeAmount
        }

        ERC20(order.baseAsset).safeTransfer(msg.sender, order.strike - feeAmount);

        return;
    }
    ..SNIP..
}

function withdrawFee(address baseAsset) public onlyOwner {
	uint256 _feeAmount = ownerFees[baseAsset];
	ownerFees[baseAsset] = 0;
	ERC20(baseAsset).safeTransfer(owner(), _feeAmount);
}
```


## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Non-Cudos Erc20 funds sent through sendToCosmos() will be lost.](https://github.com/code-423n4/2022-05-cudos-findings/issues/58) 

# Lines of code

https://github.com/code-423n4/2022-05-cudos/blob/de39cf3cd1f1e1cf211819b06d4acf6a043acda0/solidity/contracts/Gravity.sol#L595-L609


# Vulnerability details

## Impact

No checks for non-Cudos tokens mean that non-Cudos ERC20 tokens will be lost to the contract, with the user not having any chance of retrieving them.

However, the admin can retrieve them through withdrawERC20.

Impact is that users lose their funds, but admins gain them.

The mistakes could be mitigated on the contract, by checking against a list of supported tokens, so that users don't get the bad experience of losing funds and CUDOS doesn't have to manually refund users

## Proof of Concept

User sends 100 ETH through sendToCosmos, hoping to retrieve 100 synthetic ETH on Cudos chain but finds that funds never appear.


```

	function sendToCosmos(
		address _tokenContract,
		bytes32 _destination,
		uint256 _amount
	) public nonReentrant  {
		IERC20(_tokenContract).safeTransferFrom(msg.sender, address(this), _amount);
		state_lastEventNonce = state_lastEventNonce.add(1);
		emit SendToCosmosEvent(
			_tokenContract,
			msg.sender,
			_destination,
			_amount,
			state_lastEventNonce
		);
	}

```

https://github.com/code-423n4/2022-05-cudos/blob/de39cf3cd1f1e1cf211819b06d4acf6a043acda0/solidity/contracts/Gravity.sol#L595-L609

Admin can retrieve these funds should they wish, but user never gets them back because the contract does not check whether the token is supported.

```

	function withdrawERC20(
		address _tokenAddress) 
		external {
		require(cudosAccessControls.hasAdminRole(msg.sender), "Recipient is not an admin");
		uint256 totalBalance = IERC20(_tokenAddress).balanceOf(address(this));
		IERC20(_tokenAddress).safeTransfer(msg.sender , totalBalance);
	}


```




https://github.com/code-423n4/2022-05-cudos/blob/de39cf3cd1f1e1cf211819b06d4acf6a043acda0/solidity/contracts/Gravity.sol#L632-L638

## Tools Used
Logic and discussion with @germanimp

## Recommended Mitigation Steps

Add checks in sendToCosmos to check the incoming tokenAddress against a supported token list, so that user funds don't get lost and admin don't need to bother refunding.


## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-05-cudos-findings/issues/59) 

# Lines of code

https://github.com/code-423n4/2022-05-cudos/blob/de39cf3cd1f1e1cf211819b06d4acf6a043acda0/solidity/contracts/Gravity.sol#L595-L630


# Vulnerability details

## Impact

Smart contracts addresses made using the `create` opcode are deterministic based off the deployer account and the nonce of this account. 

An attacker is therefore able to predetermine the address of any smart contracts deployed using `deployERC20()`.  One of the limitations of  OpenZeppelin's `token.safeTransferFrom(from, to, amount)` is that it will succeed if there is no bytecode at the `token` address.

The impact of both of these features is that an attacker may call `sendToCosmos(_tokenContract, _destination, _amount)` for an ERC20 token before it is deployed using `deployERC20()` since the address can be calculated. The attacker may set any arbitrary amount and the `IERC20(_tokenContract).safeTransferFrom(msg.sender, address(this), _amount);` will succeed. The `SendToCosmosEvent` will be emitted and the relevant transfer will occur on the other side of the bridge.


## Proof of Concept

`IERC20(_tokenContract).safeTransferFrom(msg.sender, address(this), _amount);` succeeds if `_tokenContract` does not have any bytecode. So the attacker may call `sendToCosmos()`.

Following this the attacker calls `deployERC20()` to deploy the token contract. Note the token contract was precalculated and used as the `_tokenContract` parameter in `sendToCosmos()`.

The bridge will then process `SendToCosmosEvent` on the Cosmos chain.

```solidity
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

	function deployERC20(
		string memory _cosmosDenom,
		string memory _name,
		string memory _symbol,
		uint8 _decimals
	) public {
		// Deploy an ERC20 with entire supply granted to Gravity.sol
		CosmosERC20 erc20 = new CosmosERC20(address(this), _name, _symbol, _decimals);

		// Fire an event to let the Cosmos module know
		state_lastEventNonce = state_lastEventNonce.add(1);
		emit ERC20DeployedEvent(
			_cosmosDenom,
			address(erc20),
			_name,
			_symbol,
			_decimals,
			state_lastEventNonce
		);
	}
```

## Recommended Mitigation Steps

This issue may be resolved by enforcing `sendToCosmos()` check that `_tokenContract` contains bytecode.

```solidity
	function isContract(address _addr) private returns (bool isContract){
	    uint32 size;
	    assembly {
	        size := extcodesize(_addr)
	    }
 	    return (size > 0);
	}

	function sendToCosmos(
		address _tokenContract,
		bytes32 _destination,
		uint256 _amount
	) public nonReentrant  {
		require(isContract(_tokenContract, "Invalid contract address"));
		IERC20(_tokenContract).safeTransferFrom(msg.sender, address(this), _amount);
                ...
        }
```


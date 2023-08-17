## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-24

# [Node runner who is already known to be malicious cannot be banned before corresponding smart wallet is created](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/381) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/LiquidStakingManager.sol#L356-L377
https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/LiquidStakingManager.sol#L507-L509
https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/LiquidStakingManager.sol#L426-L492


# Vulnerability details

## Impact
Currently, the `rotateNodeRunnerOfSmartWallet` function provides the only way to set `bannedNodeRunners` to `true` for a malicious node runner. However, before the node runner calls the `registerBLSPublicKeys` function to create a smart wallet, calling the `rotateNodeRunnerOfSmartWallet` function reverts. This means that for a node runner, who is already known to be malicious such as someone controlling a hacker address, calling the `isNodeRunnerBanned` function always return `false` before the `registerBLSPublicKeys` function is called for the first time, and executing `require(isNodeRunnerBanned(msg.sender) == false, "Node runner is banned from LSD network")` when calling the `registerBLSPublicKeys` function for the first time is not effective. As the monitoring burden can be high, the malicious node runner could interact with the protocol maliciously for a while already after the `registerBLSPublicKeys` function is called until the DAO notices the malicious activities and then calls the `rotateNodeRunnerOfSmartWallet` function. When the DAO does not react promptly, some damages to the protocol could be done already.

https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/LiquidStakingManager.sol#L356-L377
```solidity
    function rotateNodeRunnerOfSmartWallet(address _current, address _new, bool _wasPreviousNodeRunnerMalicious) external {
        ...

        if (msg.sender == dao && _wasPreviousNodeRunnerMalicious) {
            bannedNodeRunners[_current] = true;
            emit NodeRunnerBanned(_current);
        }

        ...
    }
```

https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/LiquidStakingManager.sol#L507-L509
```solidity
    function isNodeRunnerBanned(address _nodeRunner) public view returns (bool) {
        return bannedNodeRunners[_nodeRunner];
    }
```

https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/LiquidStakingManager.sol#L426-L492
```solidity
    function registerBLSPublicKeys(
        bytes[] calldata _blsPublicKeys,
        bytes[] calldata _blsSignatures,
        address _eoaRepresentative
    ) external payable nonReentrant {
        ...
        require(isNodeRunnerBanned(msg.sender) == false, "Node runner is banned from LSD network");

        address smartWallet = smartWalletOfNodeRunner[msg.sender];

        if(smartWallet == address(0)) {
            // create new wallet owned by liquid staking manager
            smartWallet = smartWalletFactory.createWallet(address(this));
            emit SmartWalletCreated(smartWallet, msg.sender);

            // associate node runner with the newly created wallet
            smartWalletOfNodeRunner[msg.sender] = smartWallet;
            nodeRunnerOfSmartWallet[smartWallet] = msg.sender;

            _authorizeRepresentative(smartWallet, _eoaRepresentative, true);
        }

        ...
    }
```

## Proof of Concept
Please add the following test in `test\foundry\LSDNFactory.t.sol`. This test will pass to demonstrate the described scenario.
```solidity
    function testMaliciousNodeRunnerCannotBeBannedBeforeCorrespondingSmartWalletIsCreated() public {
        vm.prank(address(factory));
        manager.updateDAOAddress(admin);

        // Simulate a situation where accountOne is known to be malicious already.
        // accountOne is not banned at this moment.
        assertEq(manager.bannedNodeRunners(accountOne), false);

        // Calling the rotateNodeRunnerOfSmartWallet function is the only way to ban accountOne;
        //   however, calling it reverts because accountOne has not called the registerBLSPublicKeys function to create a smart wallet yet.
        // This means that it is not possible to prevent accountOne from interacting with the protocol until her or his smart wallet is created.
        vm.prank(admin);
        vm.expectRevert("Wallet does not exist");
        manager.rotateNodeRunnerOfSmartWallet(accountOne, accountTwo, true);
    }
```

## Tools Used
VSCode

## Recommended Mitigation Steps
A function, which should be only callable by the DAO, that can directly set `bannedNodeRunners` for a node runner can be added.
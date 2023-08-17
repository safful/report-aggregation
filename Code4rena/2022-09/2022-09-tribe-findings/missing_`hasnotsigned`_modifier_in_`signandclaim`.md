## Tags

- bug
- disagree with severity
- high quality report
- primary issue
- QA (Quality Assurance)
- sponsor confirmed
- edited-by-warden

# [Missing `hasNotSigned` modifier in `signAndClaim`](https://github.com/code-423n4/2022-09-tribe-findings/issues/51) 

# Lines of code

https://github.com/code-423n4/2022-09-tribe/blob/769b0586b4975270b669d7d1581aa5672d6999d5/contracts/shutdown/fuse/RariMerkleRedeemer.sol#L93


# Vulnerability details

## Impact
[This](https://github.com/code-423n4/2022-09-tribe/blob/769b0586b4975270b669d7d1581aa5672d6999d5/contracts/shutdown/fuse/MultiMerkleRedeemer.sol#L41) comment and existence of the [`testCannotSignTwice`](https://github.com/code-423n4/2022-09-tribe/blob/769b0586b4975270b669d7d1581aa5672d6999d5/contracts/test/integration/shutdown/fuse/rariMerkleRedeemer.t.sol#L453) test case makes it clear that intended behavior of the protocol is to prevent users from submitting a signature of `MESSAGE_HASH` twice.

However, any user can circumvent this and overwrite once provided signature with another valid one.

## Proof of Concept
Add the following test case to [this file](https://github.com/code-423n4/2022-09-tribe/blob/main/contracts/test/integration/shutdown/fuse/rariMerkleRedeemer.t.sol) and run integration tests:

```
    function testCanSignTwice() public {
        vm.startPrank(addresses[0]);

        IERC20(cToken0).approve(address(redeemer), 100_000_000e18);
        (uint8 v0, bytes32 r0, bytes32 s0) = vm.sign(keys[0], redeemer.MESSAGE_HASH());

        bytes memory signature0 = bytes.concat(r0, s0, bytes1(v0));

        redeemer.sign(signature0);

        address[] memory cTokens;
        uint256[] memory amounts;
        bytes32[][] memory merkleProofs;

        // vm.expectRevert("User has already signed");
        redeemer.signAndClaim(signature0, cTokens, amounts, merkleProofs);

        vm.stopPrank();
    }
```
As we can see, `signAndClaim` doesn't revert despite non-zero `userSignatures[msg.sender]` because of the missing `hasNotSigned` modifier. Moreover, when arguments passed to `redeemer.signAndClaim` are a signature and empty lists, then this function effectively behaves just like `_sign`.


## Tools Used
Foundry

## Recommended Mitigation Steps
Add missing `hasNotSigned` modifier to the `redeemer.signAndClaim`  function.
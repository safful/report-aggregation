## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- valid

# [`Project.raiseDispute()` doesn't use approvedHashes - meaning users who use contracts can't raise disputes](https://github.com/code-423n4/2022-08-rigor-findings/issues/340) 

# Lines of code

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L493-L536


# Vulnerability details


## Impact
In case users are using a contract (like a multisig wallet) to interact with a project, they can't raise a dispute.

The sponsors have added the `approveHash()` function to support users who wish to use contracts as builder/GC/SC. However, the `Project.raiseDispute()` function doesn't check them, meaning if any of those users wish to raise a dispute they can't do it.

## Proof of Concept
I've modified [the following test](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/test/utils/disputeTests.ts#L179-L215), trying to use an approved hash. The test failed.

```typescript
  it('Builder can raise addTasks() dispute', async () => {
      let expected = 2;
      const actionValues = [
        [exampleHash],
        [100000000000],
        expected,
        projectAddress,
      ];
      // build and raise dispute transaction
      const [encodedData, signature] = await makeDispute(
        projectAddress,
        0,
        1,
        actionValues,
        signers[0],
        '0x4222',
      );
      const encodedMsgHash = ethers.utils.keccak256(encodedData);
      await project.connect(signers[0]).approveHash(encodedMsgHash);
      let tx = await project
        .connect(signers[1])
        .raiseDispute(encodedData, "0x");
      // expect event
      await expect(tx)
        .to.emit(disputesContract, 'DisputeRaised')
        .withArgs(1, '0x4222');
      // expect dispute raise to store info
      const _dispute = await disputesContract.disputes(1);
      const decodedAction = abiCoder.decode(types.taskAdd, _dispute.actionData);
      expect(_dispute.status).to.be.equal(1);
      expect(_dispute.taskID).to.be.equal(0);
      expect(decodedAction[0][0]).to.be.equal(exampleHash);
      expect(decodedAction[1][0]).to.be.equal(100000000000);
      expect(decodedAction[2]).to.be.equal(expected);
      expect(decodedAction[3]).to.be.equal(projectAddress);
      // expect unchanged number of tasks
      let taskCount = await project.taskCount();
      expect(taskCount).to.be.equal(expected);
    });

```

## Recommended Mitigation Steps
Make `raiseDispute()` to check for approvedHashes too
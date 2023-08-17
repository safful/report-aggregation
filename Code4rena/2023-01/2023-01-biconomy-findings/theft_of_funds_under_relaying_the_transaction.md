## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-02

# [Theft of funds under relaying the transaction](https://github.com/code-423n4/2023-01-biconomy-findings/issues/489) 

# Lines of code

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L200
https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L239
https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L248


# Vulnerability details

## Description

The `execTransaction` function is designed to accept a relayed transaction with a transaction cost refund. At the beginning of the function, the `startGas` value is calculated as the amount of gas that the relayer will approximately spend on the transaction initialization, including the base cost of `21000` gas and the cost per calldata byte of `msg.data.length * 8` gas. At the end of the function, the total consumed gas is calculated as `gasleft() - startGas` and the appropriate refund is sent to the relayer.

An attacker could manipulate the calldata to increase the refund amount while spending less gas than what is calculated by the contract. To do this, the attacker could provide calldata with zero padded bytes of arbitrary length. This would only cost 4 gas per zero byte, but the refund would be calculated as 8 gas per calldata byte. As a result, the refund amount would be higher than the gas amount spent by the relayer.

### Attack scenario

Let’s a smart wallet user signs a transaction. Some of the relayers trying to execute this transaction and send a transaction to the `SmartAccount` contract. Then, an attacker can frontrun the transaction, changing the transaction calldata by adding the zeroes bytes at the end.

So, the original transaction has such calldata:

```sol=
abi.encodeWithSignature(RelayerManager.execute.selector, (...))
```

The modified (frontrun) transaction calldata:

```sol=
// Basically, just add zero bytes at the end
abi.encodeWithSignature(RelayerManager.execute.selector, (...)) || 0x00[]
```

### PoC

The PoC shows that the function may accept the data with redundant zeroes at the end. At the code above, an attacker send a 100_000 meaningless zeroes bytes, that gives a `100_000 * 4 = 400_000` additional gas refund. Technically, it is possible to pass even more zero bytes.

```solidity=
pragma solidity ^0.8.12;

contract DummySmartWallet {
    function execTransaction(
        Transaction memory _tx,
        uint256 batchId,
        FeeRefund memory refundInfo,
        bytes memory signatures
    ) external {
        // Do nothing, just test that data with appended zero bytes are accepted by Solidity
    }
}

contract PoC {
    address immutable smartWallet;

    constructor() {
        smartWallet = address(new DummySmartWallet());
    }

    // Successfully call with original data
    function testWithOriginalData() external {
        bytes memory txCalldata = _getOriginalTxCalldata();

        (bool success, ) = smartWallet.call(txCalldata);
        require(success);
    }

    // Successfully call with original data + padded zero bytes
    function testWithModifiedData() external {
        bytes memory originalTxCalldata = _getOriginalTxCalldata();
        bytes memory zeroBytes = new bytes(100000);

        bytes memory txCalldata = abi.encodePacked(originalTxCalldata, zeroBytes);

        (bool success, ) = smartWallet.call(txCalldata);
        require(success);
    }

    function _getOriginalTxCalldata() internal pure returns(bytes memory) {
        Transaction memory transaction;
        FeeRefund memory refundInfo;
        bytes memory signatures;
        return abi.encodeWithSelector(DummySmartWallet.execTransaction.selector, transaction, uint256(0), refundInfo, signatures);
    }
}
```

## Impact

An attacker to manipulate the gas refund amount to be higher than the gas amount spent, potentially leading to arbitrary big ether loses by a smart wallet.

## Recommended Mitigation Steps

You can calculate the number of bytes used by the relayer as a sum per input parameter. Then an attacker won't have the advantage of providing non-standard ABI encoding for the PoC calldata.


```
// Sum the length of each  bynamic and static length parameters.
uint256 expectedNumberOfBytes = _tx.data.length + signatures.length + 12 * 32;
uint256 dataLen = Math.min(msg.data.length, expectedNumberOfBytes);
```

Please note, the length of the `signature` must also be bounded to eliminate the possibility to put meaningless zeroes there.
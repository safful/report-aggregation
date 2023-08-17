## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-06

# [`FeeRefund.tokenGasPriceFactor` is not included in signed transaction data allowing the submitter to steal funds](https://github.com/code-423n4/2023-01-biconomy-findings/issues/123) 

# Lines of code

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L288
https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L429-L444


# Vulnerability details

## Impact
The submitter of a transaction is paid back the transaction's gas costs either in ETH or in ERC20 tokens. With ERC20 tokens the following formula is used: $(gasUsed + baseGas) * gasPrice / tokenGasPriceFactor$. `baseGas`, `gasPrice`, and `tokenGasPriceFactor` are values specified by the tx submitter. Since you don't want the submitter to choose arbitrary values and pay themselves as much as they want, those values are supposed to be signed off by the owner of the wallet. The signature of the user is included in the tx so that the contract can verify that all the values are correct. But, the `tokenGasPriceFactor` value is not included in those checks. Thus, the submitter is able to simulate the tx with value $x$, get the user to sign that tx, and then submit it with $y$ for `tokenGasPriceFactor`. That way they can increase the actual gas repayment and steal the user's funds.

## Proof of Concept
In `encodeTransactionData()` we can see that `tokenGasPriceFactor` is not included:
```sol
    function encodeTransactionData(
        Transaction memory _tx,
        FeeRefund memory refundInfo,
        uint256 _nonce
    ) public view returns (bytes memory) {
        bytes32 safeTxHash =
            keccak256(
                abi.encode(
                    ACCOUNT_TX_TYPEHASH,
                    _tx.to,
                    _tx.value,
                    keccak256(_tx.data),
                    _tx.operation,
                    _tx.targetTxGas,
                    refundInfo.baseGas,
                    refundInfo.gasPrice,
                    refundInfo.gasToken,
                    refundInfo.refundReceiver,
                    _nonce
                )
            );
        return abi.encodePacked(bytes1(0x19), bytes1(0x01), domainSeparator(), safeTxHash);
    }
```

The value is used to determine the gas repayment in `handlePayment()` and `handlePaymentRevert()`:
```sol
    function handlePayment(
        uint256 gasUsed,
        uint256 baseGas,
        uint256 gasPrice,
        uint256 tokenGasPriceFactor,
        address gasToken,
        address payable refundReceiver
    ) private nonReentrant returns (uint256 payment) {
        // uint256 startGas = gasleft();
        // solhint-disable-next-line avoid-tx-origin
        address payable receiver = refundReceiver == address(0) ? payable(tx.origin) : refundReceiver;
        if (gasToken == address(0)) {
            // For ETH we will only adjust the gas price to not be higher than the actual used gas price
            payment = (gasUsed + baseGas) * (gasPrice < tx.gasprice ? gasPrice : tx.gasprice);
            (bool success,) = receiver.call{value: payment}("");
            require(success, "BSA011");
        } else {
            payment = (gasUsed + baseGas) * (gasPrice) / (tokenGasPriceFactor);
            require(transferToken(gasToken, receiver, payment), "BSA012");
        }
        // uint256 requiredGas = startGas - gasleft();
        //console.log("hp %s", requiredGas);
    }
```

That's called at the end of `execTransaction()`:
```sol
            if (refundInfo.gasPrice > 0) {
                //console.log("sent %s", startGas - gasleft());
                // extraGas = gasleft();
                payment = handlePayment(startGas - gasleft(), refundInfo.baseGas, refundInfo.gasPrice, refundInfo.tokenGasPriceFactor, refundInfo.gasToken, refundInfo.refundReceiver);
                emit WalletHandlePayment(txHash, payment);
            }
```

As an example, given that:
- `gasUsed = 1,000,000`
- `baseGas = 100,000`
- `gasPrice = 10,000,000,000` (10 gwei)
- `tokenGasPriceFactor = 18`

You get $(1,000,000 + 100,000) * 10,000,000,000 / 18 = 6.1111111e14$. If the submitter executes the transaction with `tokenGasPriceFactor = 1` they get $1.1e16$ instead, i.e. 18 times more.
## Tools Used
none

## Recommended Mitigation Steps
`tokenGasPriceFactor` should be included in the encoded transaction data and thus verified by the user's signature.
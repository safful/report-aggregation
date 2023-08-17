## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- sponsor confirmed
- selected for report
- M-01

# [Borrower can by mistake add own money to credit if credit is in ETH](https://github.com/code-423n4/2022-11-debtdao-findings/issues/24) 

# Lines of code

https://github.com/debtdao/Line-of-Credit/blob/audit/code4rena-2022-11-03/contracts/modules/credit/LineOfCredit.sol#L223-L244
https://github.com/debtdao/Line-of-Credit/blob/audit/code4rena-2022-11-03/contracts/utils/LineLib.sol#L59-L74


# Vulnerability details

## Impact
Borrower can by mistake add own money to credit if credit is in ETH.

## Proof of Concept
Function `LineOfCredit.addCredit` is used to create new credit.
It can be called only after contest of another party.
```solidity
    function addCredit(
        uint128 drate,
        uint128 frate,
        uint256 amount,
        address token,
        address lender
    )
        external
        payable
        override
        whileActive
        mutualConsent(lender, borrower)
        returns (bytes32)
    {
        LineLib.receiveTokenOrETH(token, lender, amount);


        bytes32 id = _createCredit(lender, token, amount);


        require(interestRate.setRate(id, drate, frate));
        
        return id;
    }
```
`LineLib.receiveTokenOrETH(token, lender, amount)` is responsible for getting payment.
https://github.com/debtdao/Line-of-Credit/blob/audit/code4rena-2022-11-03/contracts/utils/LineLib.sol#L59-L74
```solidity
    function receiveTokenOrETH(
      address token,
      address sender,
      uint256 amount
    )
      external
      returns (bool)
    {
        if(token == address(0)) { revert TransferFailed(); }
        if(token != Denominations.ETH) { // ERC20
            IERC20(token).safeTransferFrom(sender, address(this), amount);
        } else { // ETH
            if(msg.value < amount) { revert TransferFailed(); }
        }
        return true;
    }
```
As you can see in case of native token payment, `sender` is not checked to be `msg.sender`, so this makes it's possible that borrower can by mistake pay instead of lender. It sounds funny, but it's possible. What is needed is that lender call `addCredit` first and then borrower calls `addCredit` and provides value.

## Tools Used
VsCode
## Recommended Mitigation Steps
Check that if payment in ETH then `lender == msg.sender` in `addCredit` function.
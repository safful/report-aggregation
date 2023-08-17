## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- primary issue
- satisfactory
- sponsor confirmed
- selected for report
- edited-by-warden
- M-03

# [Borrower/Lender excessive ETH not refunded and permanently locked in protocol](https://github.com/code-423n4/2022-11-debtdao-findings/issues/39) 

# Lines of code

https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L292
https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L315
https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L223
https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L265
https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/utils/LineLib.sol#L71
https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L388


# Vulnerability details

## Impact
The protocol does not refund overpayment of ETH. Excessive ETH is not included in the protocols accounting as a result the funds are permanently locked in the protocol **(Loss of funds)**. 
There are multiple scenarios where excessive ETH could be sent by Borrowers and Lenders to the protocol.

The vulnerability effects at least five different scenarios and locks both the lender and borrowers ETH in LineOfCredit if overpaid. **There is no way to transfer the locked ETH back to the the users**, as the withdraw methods are dependent on accounting (which is not updated with locked ETH).

This vulnerability impacts EscrowedLine, LineOfCredit, SpigotedLine and SecuredLine

## Proof of Concept
The bug resides in `receiveTokenOrETH` function when receiving ETH. 

The function does not handle cases where `msg.value` is larger than `amount` meaning a refund is needed (`msg.value` - `amount`). In such cases, `msg.value` is added to the balance of LineOfCredit although only `amount` is used in internal accounting. Thus the excessive ETH  is permanently locked in the contract as the withdraw methods are dependent on the internal accounting.

https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/utils/LineLib.sol#L59
```
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

Scenarios where borrowers ETH funds will be locked in LineOfCredit: 
1. Borrower calls `depositAndClose` with an ETH value that is above the owed debt.
2. Borrower calls `depositAndRepay` with an ETH value that is above the amount specified in the parameters.
3. Borrower calls `close` with an ETH value that is above the owed fees.

Scenarios where lenders ETH funds will be locked in LineOfCredit:
1. Lender calls `addCredit` with and ETH value that is greater than the `amount` parameter.
2. Lender calls `increaseCredit` with and ETH value that is greater than the `amount` parameter.

The above scenarios will happen when: 
* Excessive ETH is sent with the confidence that it will be refunded (expected). Intentionally or by mistake.
* Excessive ETH will be sent (and expected to be refunded) when calling `depositeAndClose()`, `close(id)` and `depositAndRepay(amount)` as they internally update the fees with the `_accrue` method. The amount changes every second because part of the formula that calculates the fees is based on a multiplication of seconds past the previous calculations. In most cases, the caller will not know the amount of interest that will be accrued and must send excessive ETH to not revert the transaction.
    * The formula that calculates interest: 
`InterestAccrued = (rate.dRate * drawnBalance * timespan) / INTEREST_DENOMINATOR + 
(rate.fRate * (facilityBalance - drawnBalance) * timespan) / INTEREST_DENOMINATOR `
Where `timespan` is `timespan= block.timestamp - rate.lastAccrued`
    * Attached link to Debt DAO docs with more information: https://docs.debtdao.finance/faq/accrued-interest-calculation

The POC includes four of the mentioned scenarios. To run the POC add the below code to the LineOfCredit.t.sol test and execute `forge test -v`. Expected output:
```
Running 4 tests for contracts/tests/LineOfCredit.t.sol:LineTest
[PASS] test_freeze_eth_addCredit() (gas: 277920)
[PASS] test_freeze_eth_depositAndClose() (gas: 280378)
[PASS] test_freeze_eth_depositAndRepay() (gas: 302991)
[PASS] test_freeze_eth_increaseCredit() (gas: 318830)
Test result: ok. 4 passed; 0 failed; finished in 1.59ms
```
Add the following code to tests:
```
    function _addCreditEth(address token, uint256 amount) internal {
        vm.prank(borrower);
        line.addCredit(dRate, fRate, amount, token, lender);
        vm.stopPrank();
        vm.prank(lender);
        line.addCredit{value: amount}(dRate, fRate, amount, token, lender);
        vm.stopPrank();
    }
    function test_freeze_eth_depositAndClose() public {
        uint256 amount = 1 ether;
        address eth = address(0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE);

        // fund lender
        deal(lender, amount*5);
        // fund borrower
        deal(borrower, amount*5);

        // add credit to line
        _addCreditEth(eth, amount);

        //borrow 1 ether
        bytes32 id = line.ids(0);
        vm.startPrank(borrower);
        line.borrow(id, amount);
        vm.stopPrank();
        
        //depositAndClose full extra funds (amount * 2)
        vm.startPrank(borrower);
        line.depositAndClose{value:amount*2}();
        vm.stopPrank();

        //validate funds are stuck
        console.log(address(line).balance);
        assert(address(line).balance == amount*2 - amount);
    }

     function test_freeze_eth_depositAndRepay() public {
        uint256 amount = 1 ether;
        address eth = address(0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE);

        // fund lender
        deal(lender, amount*5);
        // fund borrower
        deal(borrower, amount*5);

        // add credit to line
        _addCreditEth(eth, amount);

        //borrow 1 ether
        bytes32 id = line.ids(0);
        vm.startPrank(borrower);
        line.borrow(id, amount);
        vm.stopPrank();
        
        //depositAndRepay full extra funds (amount * 2)
        vm.startPrank(borrower);
        line.depositAndRepay{value:amount*2}(amount);
        vm.stopPrank();


        // Lender calls withdraw 
        vm.startPrank(lender);
        line.withdraw(id, amount);
        vm.stopPrank();

        //validate funds are stuck
        assert(address(line).balance == amount*2 - amount);
    }

    function test_freeze_eth_addCredit() public {
        uint256 amount = 1 ether;
        address eth = address(0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE);

        // fund lender
        deal(lender, amount*5);
        // fund borrower
        deal(borrower, amount*5);

        // add credit to line
        vm.prank(borrower);
        line.addCredit(dRate, fRate, amount, eth, lender);
        vm.stopPrank();
        vm.prank(lender);
        //double msg.value then amount
        line.addCredit{value: amount*2}(dRate, fRate, amount, eth, lender);
        vm.stopPrank();

        //borrow 1 ether
        bytes32 id = line.ids(0);
        vm.startPrank(borrower);
        line.borrow(id, amount);
        vm.stopPrank();
        
        //depositAndClose full extra funds (amount)
        vm.startPrank(borrower);
        line.depositAndClose{value:amount}();
        vm.stopPrank();

        //validate funds are stuck
        assert(address(line).balance == amount*2 - amount);
    }

    function test_freeze_eth_increaseCredit() public {
        uint256 amount = 1 ether;
        address eth = address(0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE);

        // fund lender
        deal(lender, amount*5);
        // fund borrower
        deal(borrower, amount*5);

        // add credit to line
        _addCreditEth(eth, amount);
        
        // get id
        bytes32 id = line.ids(0);

        // increase credit to line
        vm.prank(borrower);
        line.increaseCredit(id, amount);
        vm.stopPrank();
        vm.prank(lender);
        //double msg.value then amount
        line.increaseCredit{value:amount*2}(id, amount);
        vm.stopPrank();

        //total amount * 3 in contract

        //borrow 2 ether
        vm.startPrank(borrower);
        line.borrow(id, amount * 2);
        vm.stopPrank();
        
        //depositAndClose full extra funds (amount)
        vm.startPrank(borrower);
        line.depositAndClose{value:amount*2}();
        vm.stopPrank();

        //validate funds are stuck
        assert(address(line).balance == amount*3 - amount*2);
    }
```

The POC demonstrates how Borrower and Lender funds get locked in the protocol.

## Tools Used

VS Code, Foundry

## Recommended Mitigation Steps
Options:
1. refund - in receiveTokenOrETH, refund tokens back to `msg.sender `if `msg.value > amount`
2. revert - change the expression `if(msg.value < amount)` to `if(msg.value != amount)` and revert the transaction.
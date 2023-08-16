## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [_notSameBlock() can be circumvented in bondToAccount() ](https://github.com/code-423n4/2021-11-malt-findings/issues/195) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function bondToAccount() of Bonding.sol has a check based on _notSameBlock()
 _notSameBlock() makes sure the same msg.sender cannot do 2 actions within the same block.

However this can be circumvented in this case:
Suppose you call bondToAccount() via a (custom) smart contract, then the msg.sender will be the address of the smart contract.
For a pseudo code proof of concept see below.

I'm not sure what the deeper reason is for the _notSameBlock() in bondToAccount().
But if it is important then circumventing this check it will pose a risk.

## Proof of Concept
call function attack1.attack()
```JS
contract attack1 {
   function attack(address account, uint256 amount) {
         call attack2.forward(account, amount);
         call any other function of malt
  }
}

contract attack2 {
   function forward(address account, uint256 amount) {
       call bonding.bondToAccount(account, amount); // uses msg.sender of attack2
   }
}
```

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/Bonding.sol#L81-L92

```JS
function bondToAccount(address account, uint256 amount) public {
    if (msg.sender != offering) {
         _notSameBlock();
    }
    ...
```

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/Permissions.sol#L135-L141
```JS
function _notSameBlock() internal {
    require( block.number > lastBlock[_msgSender()],"Can't carry out actions in the same block" );
    lastBlock[_msgSender()] = block.number;
  }
```

## Tools Used

## Recommended Mitigation Steps
Add access controls to the function bondToAccount()
An end-user could still call bond()



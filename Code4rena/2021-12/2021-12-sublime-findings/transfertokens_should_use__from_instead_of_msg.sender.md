## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [transferTokens should use _from instead of msg.sender](https://github.com/code-423n4/2021-12-sublime-findings/issues/57) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function transferTokens of SavingsAccountUtil.sol sends the excess ETH to msg.sender, while a _from parameter is also present in the function.
It seems more logical to send it to _from, like the similar function _transferTokens of Repayments.sol

Luckily in the current code the _from is always msg.sender so it doesn't pose a direct risk.
However if the code is reused or forked it might lead to unexpected issues.

Note: transferTokens and _transferTokens are very similar so they could be integrated; they have to be checked carefully when doing this

## Proof of Concept
https://github.com/code-423n4/2021-12-sublime/blob/e688bd6cd3df7fefa3be092529b4e2d013219625/contracts/SavingsAccount/SavingsAccountUtil.sol#L98-L127
```JS
function transferTokens(.... ,   address _from,   address _to ) {
     ...
       (bool success, ) = payable(address(msg.sender)).call{value: msg.value - _amount}(''); // uses msg.sender instead of _from // also uses - instead of sub

```

https://github.com/code-423n4/2021-12-sublime/blob/e688bd6cd3df7fefa3be092529b4e2d013219625/contracts/Pool/Repayments.sol#L457-L473
```JS
    function _transferTokens(  address _from,  address _to,.... ) {
       (bool refundSuccess, ) = payable(_from).call{value: msg.value.sub(_amount)}('');
```

## Tools Used

## Recommended Mitigation Steps
In function transferTokens() change msg.sender to _from



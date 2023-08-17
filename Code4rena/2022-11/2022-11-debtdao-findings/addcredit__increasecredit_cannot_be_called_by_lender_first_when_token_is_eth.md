## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- sponsor confirmed
- upgraded by judge
- selected for report
- H-03

# [addCredit / increaseCredit cannot be called by lender first when token is ETH](https://github.com/code-423n4/2022-11-debtdao-findings/issues/125) 

# Lines of code

https://github.com/debtdao/Line-of-Credit/blob/f32cb3eeb08663f2456bf6e2fba21e964da3e8ae/contracts/modules/credit/LineOfCredit.sol#L234
https://github.com/debtdao/Line-of-Credit/blob/f32cb3eeb08663f2456bf6e2fba21e964da3e8ae/contracts/modules/credit/LineOfCredit.sol#L270


# Vulnerability details

## Impact
The functions `addCredit` and `increaseCredit` both ahve a `mutualConsent` or `mutualConsentById` modifier. Furthermore, these functions are `payable` and the lender needs to send the corresponding ETH with each call. However, if we look at the mutual consent modifier works, we can a problem:
```solidity
modifier mutualConsent(address _signerOne, address _signerTwo) {
      if(_mutualConsent(_signerOne, _signerTwo))  {
        // Run whatever code needed 2/2 consent
        _;
      }
}

function _mutualConsent(address _signerOne, address _signerTwo) internal returns(bool) {
        if(msg.sender != _signerOne && msg.sender != _signerTwo) { revert Unauthorized(); }

        address nonCaller = _getNonCaller(_signerOne, _signerTwo);

        // The consent hash is defined by the hash of the transaction call data and sender of msg,
        // which uniquely identifies the function, arguments, and sender.
        bytes32 expectedHash = keccak256(abi.encodePacked(msg.data, nonCaller));

        if (!mutualConsents[expectedHash]) {
            bytes32 newHash = keccak256(abi.encodePacked(msg.data, msg.sender));

            mutualConsents[newHash] = true;

            emit MutualConsentRegistered(newHash);

            return false;
        }

        delete mutualConsents[expectedHash];

        return true;
}
```
The problem is: On the first call, when the other party has not given consent to the call yet, the modifier does not revert. It sets the consent of the calling party instead.

This is very problematic in combination with sending ETH for two reasons:
1.) When the lender performs the calls first and sends ETH along with the call, the call will not revert. It will instead set the consent for him, but the sent ETH is lost.
2.) Even when the lender thinks about this and does not provide any ETH on the first call, the borrower has to perform the second call. Of course, he will not provide the ETH with this call, but this will cause the transaction to revert. There is now no way for the borrower to also grant consent, but still let the lender perform the call.

## Proof Of Concept
Lender Alice calls `LineOfCredit.addCredit` first to add a credit with 1 ETH. She sends 1 ETH with the call. However, because borrower Bob has not performed this call yet, the function body is not executed, but the 1 ETH is still sent. Afterwards, Bob wants to give his consent, so he performs the same call. However, this call reverts, because Bob does not send any ETH with it. 

## Recommended Mitigation Steps
Consider implementing an external function to grant consent to avoid this scenario. Also consider reverting when ETH is sent along, but the other party has not given their consent yet.
## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [Claim airdrop repeatedly](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/129) 

# Handle

gpersoon


# Vulnerability details

## Impact
Suppose someone claims the last part of his airdrop via claimExact() of AirdropDistribution.sol
Then airdrop[msg.sender].amount will be set to 0.

Suppose you then call validate() again. 
The check "airdrop[msg.sender].amount == 0" will allow you to continue, because amount has just be set to 0.
In the next part of the function, airdrop[msg.sender] is overwritten with fresh values and airdrop[msg.sender].claimed will be reset to 0.

Now you can claim your airdrop again (as long as there are tokens present in the contract)

Note: The function claim() prevents this from happening via "assert(airdrop[msg.sender].amount - claimable != 0);", which has its own problems, see other reported issues.

## Proof of Concept
// https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/vesting/contracts/AirdropDistribution.sol#L555-L563

function claimExact(uint256 _value) external nonReentrant {
        require(msg.sender != address(0));
        require(airdrop[msg.sender].amount != 0);
        
        uint256 avail = _available_supply();
        uint256 claimable = avail * airdrop[msg.sender].fraction / 10**18; //
        if (airdrop[msg.sender].claimed != 0){
            claimable -= airdrop[msg.sender].claimed;
        }

        require(airdrop[msg.sender].amount >= claimable); // amount can be equal to claimable
        require(_value <= claimable);                       // _value can be equal to claimable
        airdrop[msg.sender].amount -= _value;      // amount will be set to 0 with the last claim


// https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/vesting/contracts/AirdropDistribution.sol#L504-L517
function validate() external nonReentrant {
        ...
        require(airdrop[msg.sender].amount == 0, "Already validated.");
        ...
             Airdrop memory newAirdrop = Airdrop(airdroppable, 0, airdroppable, 10**18 * airdroppable / airdrop_supply);
             airdrop[msg.sender] = newAirdrop;
             validated[msg.sender] = 1;   // this is set, but isn't checked on entry of this function

## Tools Used

## Recommended Mitigation Steps
Add the following to validate() :
        require(validated[msg.sender]== 0, "Already validated.");


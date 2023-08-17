## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- old-submission-method

# [Early contributor can always become majority of crowdfund leading to rugging risks.](https://github.com/code-423n4/2022-09-party-findings/issues/284) 

# Lines of code

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/crowdfund/BuyCrowdfundBase.sol#L114-L135


# Vulnerability details

## Description
Voting power is distributed to crowdfund contributors according to the amount contributed divided by NFT purchase price. Attacker can call the buy() function of BuyCrowdfund / CollectionBuyCrowdfund, and use only the first X amount of contribution from the crowdfund, such that attacker's contribution > X/2. He will pass his contract to the buy call, which will receive X and will need to add some additional funds, to purchase the NFT. If the purchase is successful, attacker will have majority rule in the created party. If the party does not do anything malicious, this is a losing move for attacker, because the funds they added on top of X to compensate for the NFT price will eventually be split between group members. However, with majority rule there are various different exploitation vectors attacker may use to steal the NFT from the party ( detailed in separate reports). Because it is accepted that single actor majority is dangerous, but without additional vulnerabilities attacker cannot take ownership of the party's assets, I classify this as a medium. The point is that users were not aware they could become minority under this attack flow.


## Impact
Early contributor can always become majority of crowdfund leading to rugging risks.

## Proof of Concept
1. Victim A opens BuyCrowdfund and deposits 20 ETH
2. Attacker deposits 30 ETH
3. Victim B deposits 50 ETH
4. Suppose NFT costs 100 ETH
5. Attacker will call buy(), requesting 59ETH buy price. His contract will add 41 additional ETH and buy the NFT.
6. Voting power distributed will be: 20 / 59 for Victim A, 30 / 59 for Attacker, 9 / 59 for Victim B. Attacker has majority.
7. User can use some majority attack to take control of the NFT, netting 100 (NFT value) - 41 (external contribution) - 30 (own contribution) = 29 ETH

	
## Tools Used
Manual audit.

## Recommended Mitigation Steps
Add a Crowdfund property called minimumPrice, which will be visible to all. Buy() function should not accept NFT price < minimumPrice. Users now have assurances that are not susceptible to majority rule if they deposited enough ETH below the minimumPrice.


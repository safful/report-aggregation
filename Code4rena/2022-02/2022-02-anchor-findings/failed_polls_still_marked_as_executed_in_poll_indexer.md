## Tags

- bug
- QA (Quality Assurance)
- disagree with severity
- sponsor confirmed

# [Failed polls still marked as executed in poll indexer](https://github.com/code-423n4/2022-02-anchor-findings/issues/4) 

# Lines of code

https://github.com/code-423n4/2022-02-anchor/blob/7af353e3234837979a19ddc8093dc9ad3c63ab6b/contracts/anchor-token-contracts/contracts/gov/src/contract.rs#L530-L531


# Vulnerability details

# Impact
Users will still be able to find failed polls under "Executed" and "Failed"

# Proof of Concept
A poll attempts to execute and fails. That poll will stay in the poll indexer as executed and failed.

# Mitigation
`Passed` should be `Executed`. 

Test proof: https://pastebin.com/Cz0wujn9



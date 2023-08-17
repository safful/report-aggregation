## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Lend method signature for illuminate does not track the accumulated fee ](https://github.com/code-423n4/2022-06-illuminate-findings/issues/208) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Lender.sol#L185-L235


# Vulnerability details

Normally the amount of fees after ```calculateFee``` should be added into ```fees[u]``` so that the admin could withdraw it through ```withdrawFee```. However, illuminate ledning does not track ```fees[u]```. Therefore, the only way to get fees back is through ```withdraw``` which admin needs to wait at least 3 days before receiving the fees.  

### Mitigations 
Add the amount of fee after each transaction into ```fees[u]``` like other lending method.  
for example: ``` fees[u] += calculateFee(a);```


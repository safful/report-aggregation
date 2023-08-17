## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Division Before Multiplication Can Lead To Zero Rounding Of Return Amount](https://github.com/code-423n4/2022-06-illuminate-findings/issues/48) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/92cbb0724e594ce025d6b6ed050d3548a38c264b/lender/Lender.sol#L280


# Vulnerability details

## Impact

There is a division before multiplication bug that exists in [`lend()`](https://github.com/code-423n4/2022-06-illuminate/blob/92cbb0724e594ce025d6b6ed050d3548a38c264b/lender/Lender.sol#L280) for the Swivel case.

If `order.premium` is less than `order.principal` then `returned` will round to zero due to the integer rounding.

When this occurs the user's funds are essentially lost. That is because they transfer in the underlying tokens but the amount sent to `yield(u, y, returned, address(this))` will be zero.

## Proof of Concept

```solidity
    function lend(
        uint8 p,
        address u,
        uint256 m,
        uint256[] calldata a,
        address y,
        Swivel.Order[] calldata o,
        Swivel.Components[] calldata s
    ) public unpaused(p) returns (uint256) {

        // lent represents the number of underlying tokens lent
        uint256 lent;
        // returned represents the number of underlying tokens to lend to yield
        uint256 returned;

        {
            uint256 totalFee;
            // iterate through each order a calculate the total lent and returned
            for (uint256 i = 0; i < o.length; ) {
                Swivel.Order memory order = o[i];
                // Require the Swivel order provided matches the underlying and maturity market provided
                if (order.underlying != u) {
                    revert NotEqual('underlying');
                } else if (order.maturity > m) {
                    revert NotEqual('maturity');
                }
                // Determine the fee
                uint256 fee = calculateFee(a[i]);
                // Track accumulated fees
                totalFee += fee;
                // Sum the total amount lent to Swivel (amount of ERC5095 tokens to mint) minus fees
                lent += a[i] - fee;
                // Sum the total amount of premium paid from Swivel (amount of underlying to lend to yield)
                returned += (a[i] - fee) * (order.premium / order.principal);

                unchecked {
                    i++;
                }
            }
            // Track accumulated fee
            fees[u] += totalFee;

            // transfer underlying tokens from user to illuminate
            Safe.transferFrom(IERC20(u), msg.sender, address(this), lent);
            // fill the orders on swivel protocol
            ISwivel(swivelAddr).initiate(o, a, s);

            yield(u, y, returned, address(this));
        }

        emit Lend(p, u, m, lent);
        return lent;
    }
```

Specifically the function `returned += (a[i] - fee) * (order.premium / order.principal);`

## Recommended Mitigation Steps

The multiplication should occur before division, that is `((a[i] - fee) * order.premium) / order.principal);`.


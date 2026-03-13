---
severity: [L-1]
status: Pending
affected-contracts: TSwapPool.sol
---
**Title:**

The protocol is only handling 18 decimals tokens

**Description:**

If a pool is created for a token with less or more than 18 decimals, the 2 price getter functions `getPriceOfOneWethInPoolTokens` and `getPriceOfOnePoolTokenInWeth` will return wrong values.

**Recommended Mitigation:**

Use the decimals function of the ERC-20 interface.
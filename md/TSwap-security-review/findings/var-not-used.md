---
severity: [G-2]
status: Pending
affected-contracts: TSwapPool.sol
---
**Title:**

The `poolTokenReserves` local variable is declared but never used in `deposit` function

**Description:**

This variable is useless and was probably created for documentation. It can be explained in comments to avoid consuming gas for no reason.

**Recommended Mitigation:**

Delete the variable:

```diff
-    uint256 poolTokenReserves = i_poolToken.balanceOf(address(this));
```
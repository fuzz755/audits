---
severity: [M-1]
status: Pending
affected-contracts: TSwapPool.sol
---
**Title:**

The `deadline` parameter is declared but never used in the `deposit` function

**Description:**

Using a deadline on swaps is a safety precaution to avoid trades to be executed at unpredicatable price changes. If the deadline is reached without the swap being executed, the transaction should revert and let the user choose if he wants to try again at the new price.
The `deadline` is declared as a parameter inside the `deposit` function but it is never used.

**Impact:**

Unfavorable prices for users.

**Recommended Mitigation:**

Use the deadline check modifier:

```diff
function deposit(
    uint256 wethToDeposit,
    uint256 minimumLiquidityTokensToMint,
    uint256 maximumPoolTokensToDeposit,
    uint64 deadline
)
    external
    revertIfZero(wethToDeposit)
+    revertIfDeadlinePassed(deadline)
    returns (uint256 liquidityTokensToMint)
```
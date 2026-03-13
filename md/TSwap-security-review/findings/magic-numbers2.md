---
severity: [I-3]
status: Pending
affected-contracts: TSwapPool.sol
---
**Title:**

Use of magic numbers

**Description:**

The `getInputAmountBasedOnOutput` function is calculating a local variable using magic numbers to process fees: `997` and `10000`. These numbers should be constants and have a proper name describing what they do.

**Impact:**

Using magic numbers is often leading to mistakes, just like here: it should be `1000` not `10000`.

**Recommended Mitigation:**

Create constant variables in the contract storage:

```diff
    uint256 private constant MINIMUM_WETH_LIQUIDITY = 1_000_000_000;
    uint256 private swap_count = 0;
    uint256 private constant SWAP_COUNT_MAX = 10;
+    // divide FEE by FEE_PRECISION to get the true fee ratio
+    uint256 private constant FEE = 997;
+    uint256 private constant FEE_PRECISION = 1000;

...

-    ((inputReserves * outputAmount) * 10000) /
-    ((outputReserves - outputAmount) * 997);
+    ((inputReserves * outputAmount) * FEE_PRECISION) /
+    ((outputReserves - outputAmount) * FEE);
```
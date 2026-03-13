---
severity: [H-2]
status: Pending
affected-contracts: TSwapPool.sol
---
**Title:**

Wrong formula leads to fee calculation error and loss of funds for users

**Description:**

The `getInputAmountBasedOnOutput` function is used in `swapExactOutput` to calculate the amount of input tokens to pay in order to get a certain amount of output tokens. However, the return formula is wrong, it is multiplying the final output by `10000` instead of `1000`. Resulting in a return value 10 times higher than it should be.

**Impact:**

Fees are 90.03% instead of 0.3%. Swappers will pay way more fees than expected to liquidity providers.

**Recommended Mitigation:**

Fix the formula:

```diff
    return
-        ((inputReserves * outputAmount) * 10000) /
+        ((inputReserves * outputAmount) * 1000) /
        ((outputReserves - outputAmount) * 997);
```
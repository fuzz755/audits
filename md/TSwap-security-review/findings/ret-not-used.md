---
severity: [G-4]
status: Pending
affected-contracts: TSwapPool.sol
---
**Title:**

The `swapExactInput` function always returns zero

**Description:**

The `swapExactInput` function is returning a variable named `output`. However, this variable is never used inside the function. Because of this, the function will always return zero. The intended behavior is probably to return the output amount which is named `outputAmount`.

**Recommended Mitigation:**

Rename the return variable:

```diff
    function swapExactInput(
        IERC20 inputToken,
        uint256 inputAmount,
        IERC20 outputToken,
        uint256 minOutputAmount,
        uint64 deadline
    )
        public
        revertIfZero(inputAmount)
        revertIfDeadlinePassed(deadline)
-        returns (uint256 output)
+        returns (uint256 outputAmount)
    {

...

-        uint256 outputAmount = getOutputAmountBasedOnInput(
+        outputAmount = getOutputAmountBasedOnInput(
            inputAmount,
            inputReserves,
            outputReserves
        );
```
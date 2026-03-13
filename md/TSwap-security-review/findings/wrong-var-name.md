---
severity: [M-3]
status: Pending
affected-contracts: TSwapPool.sol
---
**Title:**

The `sellPoolTokens` function's name is misleading and not working as intended

**Description:**

The wrapper function `sellPoolTokens` has been created to be a helper function, that will let user sell pool tokens into WETH with a single parameter `poolTokenAmount`. If the user wants to sell tokens, the input should be the pool token and the output should be WETH. However, the `sellPoolTokens` function is calling this function:

```javascript
    swapExactOutput(
        i_poolToken,
        i_wethToken,
        poolTokenAmount,
        uint64(block.timestamp)
    );
```

This function allows users to swap by choosing the amount of output tokens to receive, but it is taking `poolTokenAmount` as a parameter, which is our input token.
If we want to use the amount of pool tokens as a parameter, we need to use `swapExactInput` instead.

**Impact:**

Users will sell the wrong amount of pool tokens if they call the function.

**Recommended Mitigation:**

Change the function called inside of `sellPoolTokens`:
```javascript
    function sellPoolTokens(
        uint256 poolTokenAmount
    ) external returns (uint256 wethAmount) {
        return
            swapExactInput(
                i_poolToken,
                poolTokenAmount,
                i_wethToken,
                0
                uint64(block.timestamp)
            );
    }
```
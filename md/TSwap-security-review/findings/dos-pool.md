---
severity: [H-1]
status: Pending
affected-contracts: TSwapPool.sol
---
**Title:**

DoS and permanent loss of deposits in pools

**Description:**

When a user calls the `TSwapPool::deposit` function, they can use the parameter `maximumPoolTokensToDeposit` to chose how many pool tokens they will deposit at most (corresponding to the amount of WETH). When this happens, we have 2 possible cases:
1) The pool already has liquidity: in this case, the `maximumPoolTokensToDeposit` variable is protecting the user from sending too much pool tokens to the pool.
2) The pool has no liquidity: in this case, the `maximumPoolTokensToDeposit` variable is used to define the initial amount of pool tokens inside of the pool.
The issue is related to the second case. A user may set the parameter to zero, and send a minimum amount of WETH in the pool. By doing so, the pool will have `X=0` pool tokens and `Y` WETH.
This means that the pool token has an infinite price and we will probably have a zero division somewhere. But there is a simpler problem: users will never be able to withdraw from a pool with `X=0` pool tokens. Why? Look at this snippet from the `withdraw` function:
```
if (poolTokensToWithdraw < minPoolTokensToWithdraw) {
    revert TSwapPool__OutputTooLow(
        poolTokensToWithdraw,
        minPoolTokensToWithdraw
    );
}
```
The function is taking a `minPoolTokensToWithdraw` parameter to let users chose how many pool tokens they except to receive. If the pool has 0 pool tokens to withdraw, we will be in a situation where the condition `poolTokensToWithdraw < minPoolTokensToWithdraw` is always true except when `minPoolTokensToWithdraw == 0`.
However the function is doing a zero check:
```
revertIfZero(minPoolTokensToWithdraw)
```
If the `minPoolTokensToWithdraw` parameter is set to `0`, the transaction will revert because of this.

**Impact:**

Liquidity providers' WETH are permanently lost, and the pool is not usable.

**Recommended Mitigation:**

Do a zero check on deposits:

```diff
    function deposit(
        uint256 wethToDeposit,
        uint256 minimumLiquidityTokensToMint,
        uint256 maximumPoolTokensToDeposit,
        uint64 deadline
    )
        external
        revertIfZero(wethToDeposit)
+       revertIfZero(maximumPoolTokensToDeposit)
        returns (uint256 liquidityTokensToMint)
```
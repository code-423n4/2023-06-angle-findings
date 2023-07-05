# QA

## `DiamondProxy` License
`DiamondProxy` is a fork of https://github.com/mudgen/diamond-3/blob/master/contracts/Diamond.sol, but has changed the license from MIT to CC0-1.0. While this should be no problem legally (as far as I know), there is no reason to change the license in my opinion.
## Nested Loop in `Redeemer._redeem`
The function `_redeem` iterates over all tokens. Moreover, the check `LibHelpers.checkList(tokens[i], forfeitTokens)` (which is linear in the length of `forfeitTokens`) is performed within loop iteration. Therefore if there are $n$ tokens and $m$ tokens to be forfeited, the overall complexity will be $O(n*m)$. This can be very expensive depending on the values of $n$ and $m$. As an alternative, the forfeited tokens could be stored in a mapping in the beginning and this mapping could be queried (constant complexity) within each iteration.
## `RewardHandler.sellRewards` requires trust in seller
There is a check within `RewardHandler.sellRewards` to confirm that the balance of collateral assets has increased. However, this check does not change the fact that trust in the seller is required. A malicious seller could still craft an order with huge slippage that sells tokens at a very poor rate and only increases collateral slightly. An alternative would be to construct the 1inch payload dynamically instead of relying on the user to provide it.
## Reliance on `decimals()`
`LibSetters.addCollateral` queries `decimals()` and therefore relies on the fact that tokens implement this function. While this is not guaranteed and generally against EIP 20 ("OPTIONAL - This method can be used to improve usability, but interfaces and other contracts MUST NOT expect these values to be present."), it should not be a huge problem in practice because most ERC20 tokens implement the functions. Nevertheless, because the collateral is provided by the user anyways, an (optional) alternative would be to let him also provide the decimals.
## (Temporary) manipulations of `IManager.totalAssets()` would break the system
One potential attack vector for the system is (temporarily) manipulating the return value of `IManager.totalAssets()`. Depending on how the strategies are implemented, it may be possible to inflate their values, for instance by using flash loans and temporarily depositing / withdrawing. Because no strategies were in-scope, it is not possible to judge if this is currently possible, but it is something to keep in mind (and potentially document) when building strategies.
## Third Order Taylor Approximation can be imprecise
The `Savings` contract uses a third-order taylor approximation for approximating $(1 + i)^n$  (i.e., continuous compounding). However, this can be imprecise for large value of $ni$. For instance, consider the difference when the assets are 10,000, $i = 0.01$ , $n = 10$:
```math
10000*(1.01)^{10} - 10000*(1+0.01*10+10*9*0.01^2/2+10*9*8*0.01^3/6) = 0.02
```

Now, consider the difference for $i = 0.000001$ and $n=864000$ (one week when $n$ denotes seconds):
```math
10000*(1 + 0.000001)^{864000} - 10000*(1+0.000001*864000+864000*863999*0.000001^2/2+864000*863999*863998*0.000001^3/6) = 278.2
```
An alternative would be to use a library (which usually contains higher-order terms) for the exponentiation.
## System may use prices that never existed
Becuase `LibOracle.read` combines multiple Chainlink feeds with a different timestamp, it may use a price for an asset that has never existed (when it multiplies prices with timestamp $X$ and $Y$). This has for instance happened to [Y2K in the past](https://twitter.com/spreekaway/status/1626004296146817026?lang=de). It is quite hard to avoid (one could keep multiple prices and try to find a "closest match" based on the history, but this is also not perfect), but something to keep in mind.
## Grifting possibilities for strategies
`LibSetters.revokeCollateral` performs the following check for managed collateral:
```
(, uint256 totalValue) = LibManager.totalAssets(collatInfo.managerData.config);
if (totalValue > 0) revert ManagerHasAssets();
```
Depending on the strategy used, there may be ways where another user can increase `totalAssets` (e.g., donating 1 wei), making the removal of a collateral impossible.
## Collateral may not be revokable because of rounding
`LibSetters.revokeCollateral` performs the following check:
```
if (collatInfo.decimals == 0 || collatInfo.normalizedStables > 0) revert NotCollateral();
```
However, because of the rounding within `_swap`, it can happen that `normalizedStables` is a very small value (e.g., 1 wei) and it is not possible to decrease it further.
## Swapper assumes 18 decimals for the stable coin
The `Swapper` contract assumes in multiple places that the stable coin contract has 18 decimals (e.g., [here](https://github.com/AngleProtocol/angle-transmuter/blob/196bf1035154809b1c8f454c17bb45e3745509a6/contracts/transmuter/facets/Swapper.sol#L245)). While this is true for `AgToken`'s, the system is intended to be general and usable with other underlying stablecoins according to the whitepaper. Therefore, making this configurable would make sense in my opinion.
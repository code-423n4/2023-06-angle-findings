Overall, I really like the design and the implementation of the system. The USDC depeg has shown that we need to integrate stablecoins with resilience in mind and should not simply rely on the fact that the peg holds (either explicitly by assuming a 1:1 exchange rate or implicitly by building systems that enable arbitrage opportunities when a depeg happens). Transmuter provides a very nice solution to this problem. I think it is nice that the design disincentives redemptions when the collaterization ratio is slightly below 100%, which usually results in a lot of panic, but may be only transient.
The usage of the `normalizer` variable is also a good optimization idea that reduces gas usage significantly.

## Centralization
A user of the system has to fully trust the governance address (which could be an arbitrary EOA in principle, but will presumably be a governance system). There are many sensitive actions that could be abused by the governance address to steal funds or brick the system. Some examples are:
- `Redeemer.updateNormalizer`: Setting the normalizer to a value that results in much less (or more) assets when redeeming.
- `SettersGovernor.recoverERC20`: Redeeming collateral
- `Distributor.recoverERC20`: Redeeming tokens in the distributor.

This is not necessarily a bad thing and it is understandable why this functions are provided. However, it is something that users should be aware of. Moreover, it is important that the governance system is designed such that no governance attacks are possible. For instance, if someone could acquire a majority of the governance tokens for $10M and steal $20M like that, this would be a huge risk for the protocol.

## Generalisability
The system can be used for arbitrary projects / stablecoins in principle, which is also a design goal according to the white paper. However, there are a few places where the implementation is slightly Angle-specific. For instance, the `IAgToken` interface is used for stable coin operations. While this is no huge problem, the system would be even more generalizable if it would use a more general stable coin interface or an adapter system such that other projects can write their own adapters.

## Manager Contracts
Something to keep in mind is that manager contracts (which were out of scope for this audit) can have negative security implications for the system. For instance, if a manager contract does not invest / release funds properly, they may be locked up forever.
Therefore, only because the main transmuter contracts are audited, does not mean that any concrete deployment with custom manager contracts is safe / audited.

### Time spent:
20 hours
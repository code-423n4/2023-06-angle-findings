Here will share some thoughts in 2 aspects. Composability and Penalty Factor.


# Composability

Composability is a huge advantage of DeFi, on the other side, it also could be a double sword. Considering Angle can work with other stablecoin system, and the sub-collateral is also introduced, the inter dependency could go quite complex later on. Some troublesome situation could be like: stableA use stableB as collateral, stableB use stableC as collateral, stableC again use stableA as collateral, ABC form a cycled collateral. If the cycle involves tens of tokens, it could be hard to spot the issue. But in the time of crisis, such inter dependent system will break quickly. 

The idea of sub-collateral can help to expand the available assets for the protocol. On the other hand, more attention will be needed with regards to uncontrollable external contracts. It may need special attention, such as:
- the sub-collateral changed the constitution (such as DAI and FRAX could change the backed assets).
- it may further has sub-sub-collateral.
- there is locked time period of the sub-collateral, such as notional fixed rate bond token, maybe 6 month to expiry.


The same principal applies to other parts of the contract. Overall, the protocol works great in terms of modular implementation, by which I mean the Angle system has sub-modules, acting like LEGO. One developing suggestion might be not go too far in the compatibility with other protocols, just like the sub-collateral, maybe one layer of integration is good enough, but not too much.



# Penalty Factor and Depeg Handling

The main suggestion is to add some kind of downside protection as last resort. Below is the reason.

The penalty factor can deter users from bank run, this is a clever design. But depending on the severity of the situation, the outcome might be quite different. Let's divide the users into 2 groups: "run" and "stay", and compare the 2 past big events: USDC and luna. 

1. USDC
It depegged to 0.87 and recovered afterwards. The penalty rule will work great. The "run" group will be penalized and the "stay" group will be rewarded. The most important is, USDC recovered at last. 

2. luna
It just went down to zero. At the end, everyone loses. At the beginning, the "run" group would be discouraged to exit. In the end, they ended up losing more because missing the earlier exit stop loss opportunity. 


My understanding of some main stream stable coin is, it should stay pegged, or just go zero and exit. It's hard to imagine USDC valued at $0.65 after market rally (even if the loss is too big, the admin might try to burn some of the supply to recover the price, otherwise it has nothing to do with USD). Hence, the penalty factor rule can work great in the first situation with recover, but not for collapse case. When depeg comes, the protocol will hope no one get panic and bank run, but can not assure that everything is ok if staying. It will become a difficult trade off for users.

To deal with the worst situation, some protect such as insurance can be added. With the downside protection as last resort, users wont worry that much to join the "stay" group. Worst case, if this stable coin goes to zero, they know they are still covered anyway. The last protection could act as a patch for the penalty rule.

There are some defi insurance providers, for example unslashed, nexusmutual, insurace, bridgemutual. As for the choice of this kind of protection, the cheapest tier can be used, since the collapse situation is some extremely low possible black swan. The alternative could be put option, however the choice could be limited, and the cost might be higher.

One more thing, for such kind of insurance, if try to protect USDC, the payout should not be in USDC. Because if USDC goes to 0, it it pointless to receive huge payout still in USDC.


### Time spent:
25 hours
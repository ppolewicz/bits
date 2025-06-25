# BIT-0009: Winner takes all

- **BIT Number:** 0009
- **Title:** Winner takes all
- **Author(s):** Rhef
- **Discussions-to:** [https://discord.com/channels/1120750674595024897/1384596458384134195]
- **Status:** Shelved
- **Type:** Yuma
- **Created:** 2025-06
- **Updated:** 2025-06-25
- **Requires:** 
- **Replaces:** 

## Abstract

In cooperative validation when a the validation mechanism is slightly noisy, currently used client-side winner-takes-all fails to achieve consensus, misdisributing incentive and dividends. Subtensor-side winner takes all support properly distributes incentive and dividend. The change in Yuma is very small (~5 LOC + tests).

## Motivation

The most expressive way of showing the difference between client-side WTA and server-side WTA is by using the [yuma-simulator](https://yuma-simulator.bactensor.io/) charts: TODO

WTA is not for every subnet. Only a few subnets use this approach, but for those that do, the difference between client-side and server-side is big while the implementation cost is quite small.

This feature is not meant to increase adoption of WTA, only to make it not destabilize incentive and dividends due to application of client-side WTA (which works as well as you can see on the left side of the charts).


## Specification

```python
# after calculating the dividends:
if hyperparameters[’winner-takes-all’] is True:
    idx_of_best_miner_uid = max(enumerate(nums), key=lambda t: t[1])[0]
    incentives = [0] * len(incentives)
    incentives[idx_of_best_miner_uid] = 1
```


## Rationale

This BIT was shelved before it could be fully formulated, so I'll quote discussions directly to faithfully relay the most recent state.

> ***adriansmares***: i think you're missing the point of what the paper calls server side vs client side
> 
> ***adriansmares***: if there are 2 miners which in absolute terms are scored at 99.99% and 99.98% by a validator, if you do the max (i.e. set to 1 on the validator then submit this one-off vector), you pick the first miner as the winner that takes all
> 
> ***adriansmares***: if another validator, due to whatever variance in the evaluation procedure, comes up with 99.98% and 99.99% (inversed), they pick the second miner as the winner that takes all
> 
> ***adriansmares***: the on-chain consensus now is stalled because one validator says that someone is the highest, while another one says that the other one is
>
> ***adriansmares***: whereas the proposal here is that both provide the absolute scores and then the chain does the maximisation

DT and Hudson said maybe epsilon from the last winner solves it, but lets say the miner2 thinks he's 4% better than SOTA, then two validators evaluate it:
- validator A comes up with a measured 3.99% improvement, does not move weights
- validator B comes up with a measured 4.01% improvement, moves weights
- validator C comes up with a measured 4.01% improvement, but found another model submitted by a copier with minimal changes and that one came out with 4.02%, so moved weights to miner3

Then:
- A votes on miner 1
- B votes on miner 2
- C votes on miner 3

there is no consensus and >99% of incentive is distributed to uid hoarders who didn't do any work.

Server-side WTA would tally up the results of measurements of all miners by all validators and it would make a coherent decision.


## Backwards Compatibility

None - those who don't need it just don't enable it.


## Reference Implementation (Optional)

See "Specification" section


## Security Considerations

There might be a risk with the malicious subnet owner using it to hoard incentive when root validators are against him. It must be carefully considered - probably governor should have the ability to restrict usage of this feature when it's abused.


## Copyright

This document is licensed under [The Unlicense](https://unlicense.org/).

# Constraining Central Authority: Plasma Economic Analysis

A key to decentralization is imposing constraints on authority. Constraints are used to limit undesired actions of agents in a system forcing them to engage in constructive behavior. When all agents are constrained such that incentives are aligned, trust can be established. With cryptoeconomics we design decentralized protocols which precisely define these constraints.

Plasma MVP is the first simple Plasma specification. The protocol provides scalable payments using a central operator while maintaining strong security guarantees. In this post we will use payoff matrices to identify central operators' poorly aligned incentives and address them by imposing constraints, thus recreating the Plasma MVP specification.

Let's begin!

## Unconstrained Central Payment Operator

One could naively scale payments with an unconstrained payment operator. In this scheme, users would first send ETH to a smart contract. This smart contract is completely controlled by a single operator. The operator is then tasked with storing each user's balance, updating balances when users want to send funds, and finally sending ETH to users who wish to withdraw--pretty close to how centralized exchanges and payment processors operate today.

Let's take a look at a payoff matrix in this case. There are two agents in our game: 1\) the payment operator, and 2\) a single user who wants to send payments. The payoffs are defined in the 2nd column and should be read as: 

```
( payoff_for_Operator, payoff_for_User )
```

The matrix might look something like this:

![](https://karl.tech/content/images/2018/03/Screen-Shot-2018-03-08-at-12.14.25-AM.png)

Here `social_cost` refers to the operator's reputation and any legal actions taken against them, and `user_utility` refers to how much value a user gets out of being able to transact with high throughput. If the operator is well known and lives in a jurisdiction with a good legal system, this cost could be extremely large. However, it is clear that you'd never trust an anonymous internet troll to be your operator.

Social cost alone can get us a long way, but there are some serious limitations. The first of which is that the mechanisms which establish a high `social_cost` are slow and inefficient. Building a reputation and proving that you aren't some hacker trying to steal coins is a slow and ill-defined process full of difficult social signaling and establishing relationships. Plus even when you do establish this trust, there is no way to eliminate the risk that you might make a mistake and get hacked yourself. There is a limit to how much an unconstrained central payment operator can be trusted. However, by imposing additional constraints using cryptoeconomics we can do much better.

## Adding Operator Constraints: Plasma Chain Exits

In our previous model, users had no in-protocol recourse if the payment operator were to go rogue. Plasma exits were designed to solve this problem exactly. A Plasma exit is a mechanism by which users are able to reclaim their funds \(eg. ETH\) on the mainchain even if the central operator tries to steal their money. That means if users pay attention, they will always have the upper-hand over a Plasma operator and keep their tokens safe. You can learn exactly how this can be implemented in this [Plasma MVP Overview video](https://www.youtube.com/watch?v=jTc_2tyT_lY).

![](https://karl.tech/content/images/2018/03/Screen-Shot-2018-03-08-at-12.14.33-AM.png)

With the ability for users to exit, the payoff matrix is wildly different. If users exit the chain in response to a Plasma operator misbehaving, the Plasma operator suffers a huge negative utility. The only conceivable reasons for misbehaving now are either a hack against the operator \(incentive to hurt the operator\) or the operator wanting to impose the `exit_cost` on its users, but still at the likely asymmetric cost of `all_future_fees` and `social_cost`. This is a huge step forward in the power balance between user and operator.

It is important to note the large gain in "decentralization" without the addition of any new agents. This hints that decentralization is not the number of agents in a system, but instead the distribution of power amongst the agents in that system. By imposing a constraint on the operator, we are able to reduce how much we rely on the `social_cost` of an attack to secure the network. Instead we can rely on much more straightforward incentives, which ultimately means we can trust more people to be operators. This process of establishing trust through designing constraints is, to me, what is truly exciting about blockchains.

## Addressing Additional Imbalanced Incentives

In the Plasma MVP specification outlined so far, there are still some suboptimal incentives which reduce how much trust users can place in the system. The major risk is that an operator, or someone who has hacked the operator, can still force all users to exit the chain. If the Plasma operator tries to steal someone's money by creating an invalid block, everyone's funds are instantly put at risk. This risk is so significant that everyone must immediately exit the chain and get their money back. Unfortunately, this means the operator can grief its users by imposing the `exit_cost` on everyone. The `exit_cost` largely consists of the gas cost of submitting an exit transaction to the mainchain plus the capital lockup cost of waiting a week for the dispute period to end. This gets nasty because if everyone is forced to exit at the same time, gas costs go way up and there is even a risk that the dispute period isn't long enough to fit all exit transactions. In the Plasma literature this is called the _mass exit vulnerability_.

Once again we can address this problem by imposing additional constraints on the operator. In a recent Plasma architecture, Plasma Cash, a change is made which gives each coin a unique serial number, instead of pooling coins in a single contract. With unique coin IDs, a rogue Plasma operator can only attempt to steal a single coin at a time. The risk of block invalidity which previously would have resulted in a mass exit, now has little to no effect. Each coin must be stolen individually, and can be challenged by honest users. Our mechanism can be improved further by requiring exits to contain a bond which pays the challenger if the exit is found to be invalid. This means every user that witnesses an invalid transaction is incentivized to report the misbehavior and receive a reward.

We can now adjust our model to take this change into consideration.

![](https://karl.tech/content/images/2018/03/Screen-Shot-2018-03-08-at-12.14.39-AM.png)

This is especially exciting. The only way for a user to lose their money is now the `{Attempt to steal a user's money, Passively Observe}` box. However, this outcome is easy to avoid and therefore extremely unlikely because any user is able to challenge an invalid exit, and there is a reward for catching invalid exits. All that is left is the new "withhold finalized block" which basically means the operator can decide to shut the chain down by publishing an unavailable block. However, the only effect on users is an interruption in service--no risk of funds being stolen.

With these new constraints, operators are rendered mostly harmless with no way to steal coins. We started in a world where users money is protected by someone's public reputation and a vague threat of force. We then moved into a world where users are able to retrieve their funds when an operator goes rogue. Finally, with progressively more restrictive cryptographic and economic constraints, we can guarantee the safety of tokens even with a rogue operator. One can take this even further by replacing the operator with a proof of stake voting mechanism, but I'll leave that constraint analysis for next time!

## Building the Foundations of Trust

Bold claims like 'blockchains revolutionize trust' always sounds so handwavy. Even so, these Plasma constructions really are a trust technology. Using cryptoeconomics we constructed a transparent set of constraints and incentives which, with a little analysis, can be believed to be secure. The tools we have historically used for establishing trust, eg. social credibility, the court system, are slow and inefficient. We can now begin to replace that outdated infrastructure.

The potential for what is possible if these incentives are well designed is immense. However, I am also terrified of what could happen if we get it wrong, but I'll save my rants on that for another post. For now, just know this super power exists, anyone can learn it, and I think the best way to ensure a positive future is by getting as many people involved from as many communities as we can. Let's rebuild the foundations of trust together with inclusivity, equality, and kindness in mind. Then maybe we can all live in a more inclusive, equal, and kind world.

❤️

# Plasma Cash Simple Spec

This is first pass at a full specification for a Plasma Cash chain.

_Super special thanks to Vitalik for making this post possible. And a big thanks to Joseph Poon ❤️ -- plus love to David Knott_

## Overview
Plasma Cash is a Plasma construction based around the use of unique identifiers for each token on the Plasma chain. Similar to cash in your pocket, tokens on the network are given unique serial numbers. Out of this change comes a number of benefits.

- **Sharded client-side validation** -- Clients only need to watch the Plasma chain for their tokens. That means transaction throughput can scale without increased load on individual users.
- **No confirmations** -- Transactions no longer require a two phase send plus confirmation. Instead, once a transaction is included on the main chain it can be spent.
- **Simple support for all tokens** -- There is no additional complexity adding any number distinct tokens, including non-fungible assets.
- **Minor mass exit mitigation** -- Mass exits are slightly less worrisome because a theif must submit an exit transaction for each token they wish to steal. If a chain halts tokens are still safe; however, there is still an interuption in service.

A downside is:

- **Large token denominations** -- Because each token must be assigned a serial number, one cannot mint arbatrarily small tokens. This is because at some point the gas cost of redeeming the token will be larger than the value of the token itself.

### Transaction roots
For every block in the Plasma Cash chain, a merkle root must be published to the root chain. This root can either be a merklized list, or a merkle patricia tree. In the merklized list, each index of the leaf nodes corresponds to the token ID. The values of the leaf nodes are Plasma transactions.

Transactions take the form:
```
[[prev_hash, prev_block, (target_block?), token_id, new_owner], signature]
```

A transaction spending a token with a given `token_id` is only valid if it is included in the Merkle tree at position `token_id`; that is, for each token, there is only one "place" in the Merkle tree where transactions spending that token are allowed to be. This format allows users to check the full history of the Plasma chain and prove and disprove membership for their particular tokens. This can certainly be optimized with better accumulators.

### Minting
Anyone can call a `deposit()` function of the Plasma contract, which mints a new token into existence with the amount of ether sent along with the call as the denomination; for ERC20 tokens, a `depositToken(address erc20, uint256 denomination)` call would be needed, which would pull the desired amount of tokens registered in the given ERC20 contract address and record the token type (erc20 address) and denomination.

### Spending
The spender must provide to the recipient a full history of the particular token they wish to spend. The history consists of:

* Every time a transaction spending the token was included in the Plasma chain, a Merkle proof of this must be supplied.
* For every Plasma chain block during which that particular token was not spend, a Merkle proof-of-nonexistence must be provided; that is, a Merkle branch proving that at the index in the Merkle tree corresponding to the coin there is empty data, and not a valid transaction.

The recipient then checks the token's history for any invalidity or withheld blocks. Assuming no problems, the recipient can now spend the token.

## Exits
Exits allow users to retrieve their tokens on the main chain. Invalid exits can be challenged by any user with the token's data. Each exit also contains a bond which covers the gas cost of a challenge plus a bounty for proving invalidity.

There are three types of exit challenges, they are as follows:

![Exit spent coin challenge](https://user-images.githubusercontent.com/706123/37399321-ab2b0e7e-2781-11e8-82b0-5f0a509a61c5.png)

---

![Exit Double Spend Challenge](https://user-images.githubusercontent.com/706123/37399325-af3de5fe-2781-11e8-83c5-8db932e00f0f.png)

---

![Exit With Invalid History Challenge](https://user-images.githubusercontent.com/706123/37399327-b1bea55c-2781-11e8-8f31-a06b67e53b10.png)

# Further Research

### Partial Spends
There are a number of ways to achieve partial spends. One simple solution is allow token IDs to support decimals. A token can then be broken into smaller pieces and later exited on the root chain. A drawback of this apporach is if a token is split too small, then it could once again not be worth it to withdraw.

Ideally, one could split tokens, transact on the Plasma chain, and then combine a bunch of small tokens into one large one and exit. This seems quite possible but a spec for doing so is still in the works.

Another approach worth noting is probabilistic payments. This basically means instead of paying you $0.50, I pay you $1.00 50% of the time. This can be useful for micropayments.

### Compact Spend History
Having to verify a token's history could be quite expensive if implemented poorly. It would be great if proofs of non-membership in the transaction list were easy to achieve with minimal bandwidth. Ideally one can use a zkSTARK of the history for super fast compact token history checking.

### Efficient Exit Challenges While Allowing Invalid Transactions
In the simplest version of the spec, we assume that after an invalid transaction all subsiquent transactions are invalid. Therefore, if you prove there was a single invalid state transition then the only person who can exit is the owner of the token right before the invalid transaction was included. This allows us to make the exit logic super simple.

In theory there is no problem with invalid state transitions in a token's history. Because users will validate each token's history, the user can just ignore the invalid spends. However, exit logic becomes more complex. Exits must then prove that not only is the spend valid, but that there are no invalid spends in the coin's history. There may be some nice acumulator which could help do this efficiently, but that's open research.

### Block Withholding
It would be nice if a Plasma chain can recover from unavailable blocks being added to the root chain. The problem with unavailability is if a block ever becomes available at a later date, it can invalidate honest user's transactions. This is quite scary, and it turns out data availability is one of the hardest problems in blockchain design.

To get a sense for the problem, consider the following diagram:

![Block withholding](https://user-images.githubusercontent.com/706123/37399329-b38eef40-2781-11e8-85b7-d2ba66818605.png)

A recovery method for block withholding would be nice. Blocks could be unavailable but after some mechanism ran its course, a recovery could be made without having to exit tokens back onto the main chain. It is worth noting that tokens are safe even on a Plasma chain with unavailable data. However, a mechanism like this would allow tokens to both be safe and live (transferable).

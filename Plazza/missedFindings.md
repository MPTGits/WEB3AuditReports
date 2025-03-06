## [HIGH] Issue #1 (3 dups) 1.400$ payout
BondOracleAdapter will cause massive loss of funds for a large number of bond tokens
### *Summary*
There is a chance that the dex pool we get is for liqudityToken/boundToken pair. This will result in incorrect prices as we hardcode the bound token decimals in our BoundAdapterOracle.

### *Description*

The root cause of this proble is the way Andromeda Liquidity Pools work. Whenever we have a pair (for example USDC/wstETH) the price for a token is returned in the decimals of the first token in the pair for the pool.
This would mean that in our case when we get the price for the boundToken we would either have a pool that is boundToken/LiquidityToken or liquidityToken/BoundToken. In the first case the price would be correct as the boundToken is the first token in the pair, 
in the second case though we would have problems as we are geting the price in the liquidityToken decimals, which could be different, for example the liquidityToken could be USDC which has only 6 decimals.

Then when we in the OracleReader and in the Pool we always scale the price by the BoundAdapterOracle decimals, which is always the boundToken decimals instead of the dexPool that we used. This causes the user to have great losses when withdrawing bounds as the marketPrice will be really low and
will always be used

### *Why didn't I find it?*

I didn't know how Andromeada Liquidity Pools work and that they return the price in the decimals of the first token that is part of a pair. I also didn't know that the pools are stored alphanomerically and we could get the wrong pair pool in some cases

### *Questions to ask myself so I don't miss it next time*

Are we using a Andromeda Liquidity Pool in our solution for a pair of tokens?
What is the return value and type for the reverse pair pool?
Is it possible to use the wrong pair pool?

### *Learned*

How Andromeda liquidity pool pairs work


## [HIGH] Issue #2 (3 dups) 1.400$ payout
Leverage user can avoid paying fees to bond holders by withdrawing before auction ends
### *Summary*
There are multiple similar cases that expose this issue, one of which is that levege users could avoid paying "fee" for coupons for bond holders.

### *Description*

The problem is that reserve tokens get transfered to an auction only after the auction has finisished, which means an user could track that and withdraw reserve tokens as a leverage holder
just before the auction ends. In that way he/she won't suffer a hit when the reserve of token gets lower. In this way we avoid paying fees to the bound holders. Another thing about is that creationg and reddemption
of tokens now has changes as well as the tvl of the pool changed and that is the main metric that is used for ceation and redemption.

### *Why didn't I find it?*

I didn't think of the leverage holders paying the "fee" when an auction is created to gather coupons for bound holders. Also I didn't think how this decase in tvl affects the protocol in general

### *Questions to ask myself so I don't miss it next time*

Is there any funcionality that depends on the tvl of the Pool?
Does the tvl of the Pool change in some way? How will this change affect the operations in the pool?

### *Learned*

Keep an eye for changing TVL functionality in Pools


## [MEDIUM] Issue #3 (0 dups, solo) 1.100$ payout
Bid with high price effectively can end up with lower price
### *Summary*
User can exploit arithmetics issue in the Auction when bidding to remove the lowest beidder with a worse bid

### *Description*

Users can bid with a specific kind of bid and due to an arhitmetic issue when calculating and substractiong partitions users with a "higher bid" can actaully get more reserve tokens for less coupon tokens after the caclulation is done,
and kick out the lowest bidder that actaully had a better bid for the protocol.

### *Why didn't I find it?*

It was way down in the arithmetics for a bid and I didn't spend too much time in going over the math for it

### *Questions to ask myself so I don't miss it next time*

Is there a possibility due to a precision issue to make a "worse bid" and still kick out a user with a "better bid"?

### *Learned*

N/A

## [MEDIUM] Issue #4 (0 dups, solo) 1.100$ payout
Auction date will drift irreversibly forward over time leading to loss of yield for bond holders
### *Summary*
Sneaky usage of the Sherlock rules for this issue. Since the start of an auction could be executed whnever the new distribution period has passed(90 days), we could have yield loses if we do not start it immidietly.

### *Description*

Each second/minute we are late to start a new auction would mean that we shift the next period by that time, since the lastDistribution timestamp gets assigned to current timeblock. 
At some point we would have accumulated enough time to cause yield delay and "loss" to the user. The correct way to handle this is to assign the lastDistribution to lastDistribution + distributionPeriod
instead of the current timestamp.


### *Why didn't I find it?*

Didn't focuse on this kind of interval edgecase. Also didn't consider that late payment of yield could be considered as a loss to the user.


### *Questions to ask myself so I don't miss it next time*

Is there some way to make the user repayment late?
If so is it controlable by an user or is it incorrect automatic logic? How much delay could be caused?

### *Learned*

N/A

## [MEDIUM] Issue #5 (0 dups, solo) 1.100$ payout
Approval overflow causes DoS in BalancerRouter's exitPlazaAndBalancer
### *Summary*
Really cool bug, it seems like that by default Balancer Pool Tokens have infinite allowance for their Balancer Vault.

### *Description*

Since Balancer Pool Tokens already have infinite allowance (uint256.max) then if we try to increase the allowance more we get a overflow and a revert.
In the case of Plaza this would prevent us to withdraw tokens from the PreDeposit contract when we use Balancer Tokens.


### *Why didn't I find it?*

I didn't know about this functionality of Balancer Pool Tokens


### *Questions to ask myself so I don't miss it next time*

Is there a Balancer Pool token used in the code that we try to increase allowance when sending to the balancer Pool?

### *Learned*

Balancer Pool Tokens have inifinite allowance by default for their Balancer Vault


## [HIGH] Issue #6 (4 dups) 1.000$ payout
Collateral Level Manipulation via Bondsupply
### *Summary*
We could take advantage of a sharp change of formula calculation when collateral level is above 120% and below 120%

### *Description*

The sharp change in the way we cacluclate how much leverage tokens to mint could be used to frontrun user and extract value when collateral level is close to 120%, and slightly above.
The attack scenario follows the following atack path:

1. Identify when the collateral level is slightly above 1.2
2. Calculate the exact bond supply needed to push the level below 1.2
3. Front-run user transactions with bond minting to manipulate the collateral level
4. Allow victim transactions to execute with disadvantageous rates
5. Back-run to restore the original state and extract profit
   
For example, with TVL = 1500e18, increasing bond supply from 10 to 13 can push the collateral level from 1.5 to 1.15, triggering the profitable rate switch.

This way of collaterall implementation also casuse another issue when collateral is exactly 1.2 when getting bound tokens, if we then redeem those tokens we could get a higher value than the one we purchased them from.

### *Why didn't I find it?*

Didn't spend time to investiage the difference of profit between the two formulas and if there was room to implement this kind of atack.


### *Questions to ask myself so I don't miss it next time*

Is there a way to extract profit if collaterall is around threshold?
Could I affect the collaterall around the threshold in a way so I can frontrun an user and extract value?

### *Learned*

N/A

## [HIGH] Issue #7 (6 dups) 723$ payout
More tokens can be redeemed by manipulating the collateral level
### *Summary*

Users can take advantage that when  collateral level is <= 120% boundTokens can be bought cheaper and when collateral level >=120% boundTokens price is 100$


### *Description*

Users can manipulate the collaterall level, as it calculates the "expected" collaterall level when redemming boundTokens. User can use this to make so when they redeem tokens(or they can donate to the pool since
we use the balanceOf the pool to calculate tvl) to make the collateral above 120%(if it was bellow before that) and by that redemm bound tokens at 100$. This would make the user unfairly get more tokens than expected.
The Pool should calculate the actaul collaterall level when redeeming, not the expected one.

### *Why didn't I find it?*

I didn't notice that this could cause the user to gain more funds than expected


### *Questions to ask myself so I don't miss it next time*

Is there a way to gain more funds if an action shifts collateral level from/to under/over collaterized?
How do price calculations change when shifting from under <-> over collaterized when buying/selling?

### *Learned*

Shift in collateral level could cause change in prices when redeeming/creating liquidity

## [HIGH] Issue #8 (5 dups) 723$ payout
Incorrect price returned from BondOracleAdapter contract
### *Summary*

The price is not scaled correctly


### *Description*


### *Why didn't I find it?*



### *Questions to ask myself so I don't miss it next time*


### *Learned*


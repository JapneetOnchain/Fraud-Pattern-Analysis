# Case 002: $50M Lost to a $73K Pool
## CoW Protocol Solver Routing Failure and the Compliance 
## Gaps in Intent-Based DEX Architecture

**Date:** March 12, 2026  
**Chain:** Ethereum  
**Classification:** Protocol Failure / Solver Algorithm Failure  
**Transaction Hash:** 0x9fa9feab3c1989a33424728c23e6de07a40a26a98ff7ff5139f3492ce430801f  
**Victim Wallet:** 0x98B9D979C33dD7284C854909BCC09b51FBF97Ac8  
**Solver:** 0x3980dAA7EaaD0B7e0C53cFc5c2760037270DA54D  
**Solver Execution Contract:** 0x699C5BD4D03D98dABE8EF94Ce13ba0314e4D35c8  
**Loss:** $50,395,611 (99.93% of position)  
**Recovery:** $36,389 (327.241 aEthAAVE)

---

## What Caught My Attention

This transaction was everywhere on X the day it happened. 
Everyone reported the number, $50M became $36,000, 
but nobody published the actual mechanics from raw 
transaction data. No constant product math. No internal 
transaction analysis. No explanation of why the token 
transfer sequence looks the way it does.

I pulled the transaction on Etherscan and worked through 
all 12 token transfers, the relevant internal calls, 
and the logs. What I found changed my reading of the 
case, especially the GetReserves call. That single 
internal transaction shifts the entire narrative.

---

## Summary

A whale holding $50.4M in Aave USDT attempted to switch 
their position to AAVE token using CoW Protocol's native 
Aave interface integration. The winning solver routed $38M 
worth of WETH through a SushiSwap pool with $73,000 total 
liquidity, after explicitly checking pool reserves via 
GetReserves call and routing there anyway. $50.4M became 
$36,389. This case documents the complete transaction 
sequence from raw Etherscan data, applies constant product 
mathematics to verify the loss at each hop, rules out 
deliberate manipulation, and identifies four unresolved 
compliance gaps in intent-based DEX architecture.

---

## Actors

| Actor | Address | Role |
|-------|---------|------|
| Victim | 0x98B9D979...FBF97Ac8 | Submitted swap intent |
| Solver | 0x3980dAA7...DA54D | Won CoW auction, executed tx |
| Execution Contract | 0x699C5BD4...4e4D35c8 | Solver's routing contract |
| CoW Settlement | 0x9008D19f...560ab41 | Order coordinator |
| Uniswap V3 USDT Pool | 0x4e68Ccd3...960dFa36 | First swap hop |
| SushiSwap AAVE Pool | 0xD75EA151...9df57B4 | Second swap hop, catastrophe |

---

## How CoW Protocol Works

CoW Protocol is an intent-based DEX. Users submit signed 
orders off-chain specifying what they want to swap. A network 
of competing solver bots picks up orders and finds execution 
paths. The solver offering the best output wins the auction 
and executes on-chain. CoW Protocol Settlement contract 
coordinates and settles the trade.

The victim never directly executed a transaction. They 
submitted an intent through Aave's native interface. 
A solver bot made every routing decision autonomously.

This architecture creates a critical trust dependency. 
The user has zero visibility into or control over how 
their order is executed once submitted.

---

## Transaction Sequence: 12 Token Transfers Decoded

![Image 20-03-26 at 7 42 PM](https://github.com/user-attachments/assets/ba1901fa-976a-457f-9e93-d2dbd07d12ef)

![Image 20-03-26 at 7 42 PM (1)](https://github.com/user-attachments/assets/c293fb7c-3a2b-4003-9a39-b18dd00fbebc)

Reading bottom to top, chronological order:

**Transfer 1: Interest accrued**
```
Null to Victim: 8.996581 aEthUSDT
```
Aave minted accrued interest to victim immediately before 
swap execution. Confirms victim held this Aave position 
for a period before switching. aToken interest accrues 
every block. Null address origin confirms fresh mint 
not transfer.

**Transfer 2: CoW Protocol fee**
```
Null to CoW Settlement: 0.000215 aEthUSDT
```
Tiny aEthUSDT minted directly to CoW Settlement. 
This is CoW's facilitation fee, captured as a mint 
rather than a transfer. Cross-reference against Log 9 
Trade event showing feeAmount: 0. CoW advertises zero 
fees. The fee is real but captured differently.

![Image 20-03-26 at 7 43 PM](https://github.com/user-attachments/assets/6dc49628-2a6c-4683-b948-15b1ec7fde48)

**Transfer 3: Victim submits position**
```
Victim to CoW Settlement: 50,432,688.41618 aEthUSDT
```
Entire aEthUSDT balance handed to CoW Settlement for 
execution. Principal plus accrued interest. Full position 
transferred.

**Transfer 4: Settlement to solver**
```
CoW Settlement to Execution Contract: 50,432,688.41618 aEthUSDT
```
Settlement forwards full position to solver's execution 
contract. This is the entity that will orchestrate all 
subsequent interactions.

**Transfer 5: Aave withdrawal initiated**
```
Execution Contract to Null: 50,432,688.41618 aEthUSDT burned
```
Solver burns aEthUSDT to redeem underlying USDT from Aave 
reserve. Null address destination confirms permanent 
destruction of receipt token. Aave releases underlying 
only upon confirmed burn.

**Transfer 6: USDT released**
```
Aave USDT Reserve to Execution Contract: 50,432,688.41618 USDT
```
Aave releases $50.4M USDT to solver. Aave interaction 
complete. This is the last moment the position is intact. 
$50.4M in a single contract, ready to swap. 
Everything that follows destroys it.

**Transfer 7: Uniswap output (optimistic transfer)**
```
Uniswap V3 USDT Pool to SushiSwap AAVE Pool: 17,957.81 WETH
```
Uniswap V3 sends swap output directly to SushiSwap pool, 
not back to solver. Solver pre-programmed direct 
pool-to-pool routing. Appears before Transfer 8 due to 
Uniswap V3's optimistic transfer pattern. Output sent 
first, input collected via callback after.

**Transfer 8: USDT into Uniswap**
```
Execution Contract to Uniswap V3 USDT Pool: 50,432,688.42 USDT
```
Solver sends $50.4M USDT into Uniswap V3 for first swap hop. 
Expected output at market rate: ~24,600 WETH. 
Actual output: 17,957 WETH. 
Loss at this step: ~$13.6M. Single pool routing on a 
$50M position exhausted concentrated liquidity ranges 
and pushed into increasingly thin price levels.

**Transfer 9: SushiSwap output**
```
SushiSwap AAVE Pool to Execution Contract: 331.305 AAVE ($36,904)
```
17,957 WETH worth ~$38M entered SushiSwap AAVE pool. 
331 AAVE came out. $36.77M evaporated in a single pool 
interaction against $73,000 total liquidity. 
Pool completely drained of AAVE.

**Transfer 10: AAVE deposited to Aave**
```
Execution Contract to Aave V3: 331.305 AAVE
```
Solver deposits recovered AAVE into Aave V3 to mint 
aEthAAVE receipt tokens. Final protocol interaction.

**Transfer 11: aEthAAVE minted to Settlement**
```
Null to CoW Settlement: 331.305 aEthAAVE
```
Aave mints fresh aEthAAVE to CoW Settlement. 
Null origin confirms new mint. Settlement holds 
temporarily before delivery.

**Transfer 12: Final delivery to victim**
```
CoW Settlement to Victim: 327.241 aEthAAVE ($36,389)
```
Victim receives 327.241 aEthAAVE, which is 4.064 less than 
minted in Transfer 11. The difference is solver surplus, 
CoW's compensation mechanism for the winning solver.

---

## The Math: Verified From Raw Log Data

![Image 20-03-26 at 7 43 PM](https://github.com/user-attachments/assets/7dae4319-7187-4320-bf90-cfe8b638dbea)

**Source: Log 29, SushiSwap AAVE Pool Sync Event**

Post-swap reserves confirmed from on-chain log:
```
reserve0 (AAVE): 326,666,929,169,791,895 = 0.327 AAVE
reserve1 (WETH): 17,975,464,081,898,540,030,304 = 17,975.46 WETH
```

Working backwards to pre-swap reserves:
```
AAVE before: 0.327 + 331.305 = 331.632 AAVE (~$36,940)
WETH before: 17,975.46 - 17,957.81 = 17.65 WETH (~$36,182)
Total pool liquidity: ~$73,122
```

Constant product formula verification:
```
k = 331.632 × 17.65 = 5,853.3

After adding 17,957.81 WETH:
New WETH = 17.65 + 17,957.81 = 17,975.46
New AAVE = k / 17,975.46 = 5,853.3 / 17,975.46 = 0.326 AAVE

AAVE extracted = 331.632 - 0.326 = 331.306 AAVE
```

Matches Transfer 9 exactly, 331.305 AAVE. Pool 
completely drained. Every number in this calculation 
comes directly from the Sync log with no external source.

One limitation worth noting. I calculated pre-swap 
reserves by working backwards from the post-swap Sync 
data. Without the pre-transaction state snapshot this 
introduces minor rounding. The output match to Transfer 9 
confirms the calculation is accurate within acceptable margin.

**Complete loss breakdown:**
```
Entry:     50,432,688 USDT       = $50,432,688
Uniswap:   17,957 WETH received  = $36,811,850  (-$13,620,838)
SushiSwap: 331 AAVE received     = $36,904      (-$36,774,946)
CoW fee:   4.064 aEthAAVE        = -$452
Final:     327.241 aEthAAVE      = $36,389
Total loss: $50,396,299 (99.93%)
```

---

## The Smoking Gun: Solver Had Full Information

![Image 20-03-26 at 7 39 PM](https://github.com/user-attachments/assets/7ed47564-efaa-42c5-8bde-7fb72a47088b)

Internal transaction sequence immediately before 
SushiSwap execution:
```
staticcall, Get Reserves, Solver to SushiSwap AAVE Pool
staticcall, Balance Of, Solver to WETH
staticcall, Balance Of, Solver to WETH
call, Swap, Solver to SushiSwap AAVE Pool
```

This is what changed my reading of the case.

The solver called GetReserves on the SushiSwap pool 
before routing. It retrieved the reserve data showing 
~$73,000 total liquidity. It then executed the swap anyway.

This is not a case of missing information. The solver 
had the reserve data. Its algorithm failed to apply a 
minimum liquidity threshold check before routing. 
A basic validation, if pool liquidity is less than 
order size by any reasonable safety factor then reject 
this route, would have prevented the entire loss.

The difference between a routing mistake and a routing 
failure with full information is significant. Both for 
assigning responsibility and for understanding what 
protocol controls need to exist.

---

## Accidental vs Deliberate: The Same Mechanic, Two Intents

This transaction structure is identical to a deliberate 
liquidity trap attack. Understanding the difference is 
critical for investigators.

**Accidental, what happened here:**
```
Old neglected pool (5+ years old)
No liquidity added before transaction
No liquidity removed after transaction
Solver algorithm failure with full reserve information
User ignored extraordinary slippage warning
```

**Deliberate liquidity trap, what it would look like:**
```
Attacker identifies intent-based DEX solver routing patterns
Attacker creates or selects tiny pool for target token
Attacker adds liquidity shortly before expected victim tx
Attacker manipulates solver routing toward their pool
Victim executes, attacker's pool absorbs victim's funds
Attacker removes liquidity, extracts victim's funds
```

**Five on-chain checks to distinguish accidental from deliberate:**

| Check | Accidental Signal | Deliberate Signal |
|-------|------------------|------------------|
| Pool age | Months/years old | Created days before |
| Pre-tx liquidity add | None | Mint event within 24-48hrs |
| Post-tx liquidity removal | None | Burn event within hours |
| Liquidity provider identity | Multiple LPs, old positions | Single fresh wallet |
| Solver wallet history | Legitimate bot, multiple users | Fresh wallet, single target |

The fifth check, solver wallet history, is the weakest 
signal in this table. A sophisticated attacker could 
compromise or impersonate a legitimate solver. 
The first four checks carry more investigative weight 
in practice.

**This case, all five checks confirm accidental:**
- Pool created 5 years 168 days ago ✅
- No Mint events before March 12 ✅
- No Burn events after March 12 ✅
- Multiple historical LPs ✅
- Solver has legitimate history across multiple users ✅

---

## Four Compliance Gaps in Intent-Based DEX Architecture

These gaps were not obvious from reading about the event. 
They became clear working through the raw transaction, 
specifically the attribution displacement in the token 
transfers and the fee capture mechanic in Transfer 2.

**Gap 1: Attribution displacement**

In a standard DEX swap the transaction initiator is 
the user. In CoW Protocol the on-chain transaction 
initiator is the solver bot. The victim's address only 
appears in internal logs, not as the from address 
on the transaction.

Automated tracing tools following the from address 
will trace the solver not the victim. Beneficial 
ownership requires manual reconstruction one layer 
deeper. Sophisticated actors can exploit this by 
submitting intents through CoW, letting solvers execute, 
and the on-chain trail shows bot activity not human wallet. 
Obfuscation layer without using a mixer.

**Gap 2: Solvers as unregulated intermediaries**

Solver bots execute asset swaps, collect fees, and 
route funds across protocols with zero regulatory 
obligations. No KYC. No sanctions screening. 
No AML program.

A sanctioned entity can submit a swap intent. 
A solver executes it unknowingly and earns the surplus. 
Under VARA, MiCA, and FATF guidance, whether solvers 
constitute Virtual Asset Service Providers requiring 
registration is unresolved. The regulatory gap is live.

**Gap 3: Slippage as cover for deliberate extraction**

As demonstrated in the comparative analysis above, 
accidental and deliberate liquidity trap transactions 
are structurally identical on-chain. Both produce the 
same catastrophic output. Both route through the same 
pool mechanics. Automated tools will classify both as 
legitimate swap transactions.

An investigator cannot distinguish them from token 
transfers alone. Manual reconstruction of pool history, 
liquidity provider identity, and solver wallet patterns 
is required.

**Gap 4: Protocol fee capture without regulatory status**

CoW Protocol collected fees on a $50M transaction. 
Transfer 2 shows a mint of 0.000215 aEthUSDT to CoW 
Settlement. Log 9 Trade event shows feeAmount: 0. 
CoW advertises zero fees but the fee is real, just 
structured differently to avoid the label.

No KYC on the user. No AML program. No regulatory 
registration. Protocols facilitating nine-figure asset 
swaps and collecting fees without compliance 
infrastructure represent a systemic gap that VARA 
and MiCA have not yet addressed.

Aave returned $600K in fees collected from this 
transaction. The ethical instinct was correct. 
The legal obligation remains unclear.

---

## What This Case Actually Means

**For investigators:**
This transaction is not detectable as suspicious from 
token transfers alone. The loss is only visible by 
reading the Uniswap Swap log price impact and applying 
constant product mathematics to the SushiSwap Sync log 
reserves. No analytics tool flagged this as anomalous. 
Manual decoding is the only path to understanding 
what happened.

**For compliance professionals:**
Intent-based DEX architecture introduces attribution 
gaps, unregulated intermediaries, and structurally 
identical transactions between user error and deliberate 
fraud. Current regulatory frameworks have not addressed 
any of these gaps. This will become a larger problem 
as institutional capital enters DeFi at scale.

---

## Transaction Reference

**Hash:** 0x9fa9feab3c1989a33424728c23e6de07a40a26a98ff7ff5139f3492ce430801f

**Etherscan:** https://etherscan.io/tx/0x9fa9feab3c1989a33424728c23e6de07a40a26a98ff7ff5139f3492ce430801f

# Case 002 — $50M Lost to a $73K Pool
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

## Summary

A whale holding $50.4M in Aave USDT attempted to switch their 
position to AAVE token using CoW Protocol's native Aave interface 
integration. The winning solver routed $38M worth of WETH through 
a SushiSwap pool with $73,000 total liquidity — after explicitly 
checking pool reserves via GetReserves call and routing there anyway. 
$50.4M became $36,389. This case documents the complete transaction 
sequence from raw Etherscan data, applies constant product mathematics 
to verify the loss at each hop, rules out deliberate manipulation, 
and identifies four unresolved compliance gaps in intent-based DEX 
architecture.

---

## Actors

| Actor | Address | Role |
|-------|---------|------|
| Victim | 0x98B9D979...FBF97Ac8 | Submitted swap intent |
| Solver | 0x3980dAA7...DA54D | Won CoW auction, executed tx |
| Execution Contract | 0x699C5BD4...4e4D35c8 | Solver's routing contract |
| CoW Settlement | 0x9008D19f...560ab41 | Order coordinator |
| Uniswap V3 USDT Pool | 0x4e68Ccd3...960dFa36 | First swap hop |
| SushiSwap AAVE Pool | 0xD75EA151...9df57B4 | Second swap hop — catastrophe |

---

## How CoW Protocol Works

CoW Protocol is an intent-based DEX. Users submit signed orders 
off-chain specifying what they want to swap. A network of competing 
solver bots picks up orders and finds execution paths. The solver 
offering the best output wins the auction and executes on-chain. 
CoW Protocol Settlement contract coordinates and settles the trade.

The victim never directly executed a transaction. They submitted 
an intent through Aave's native interface. A solver bot made every 
routing decision autonomously.

**This architecture creates a critical trust dependency — the user 
has zero visibility into or control over how their order is executed 
once submitted.**

---




## Transaction Sequence — 12 Token Transfers Decoded


![Image 20-03-26 at 7 42 PM](https://github.com/user-attachments/assets/336016c3-f94f-4611-bb6f-c9ada3b5e219)

![Image 20-03-26 at 7 42 PM (1)](https://github.com/user-attachments/assets/d06971de-a007-4dbc-9af4-959752f29035)

Reading bottom to top — chronological order:

**Transfer 1 — Interest accrued**
```
Null → Victim: 8.996581 aEthUSDT
```
Aave minted accrued interest to victim immediately before swap 
execution. Confirms victim held this Aave position for a period 
before switching. aToken interest accrues every block — 
Null address origin confirms fresh mint not transfer.

**Transfer 2 — CoW Protocol fee**
```
Null → CoW Settlement: 0.000215 aEthUSDT
```
Tiny aEthUSDT minted directly to CoW Settlement. 
This is CoW's facilitation fee — captured as a mint 
rather than a transfer. Cross-reference against Log 9 
Trade event showing feeAmount: 0. CoW advertises zero fees. 
The fee is real — it is just captured differently.

**Transfer 3 — Victim submits position**
```
Victim → CoW Settlement: 50,432,688.41618 aEthUSDT
```
Entire aEthUSDT balance handed to CoW Settlement for execution. 
Principal plus accrued interest. Full position transferred.

**Transfer 4 — Settlement to solver**
```
CoW Settlement → Execution Contract: 50,432,688.41618 aEthUSDT
```
Settlement forwards full position to solver's execution contract. 
This is the entity that will orchestrate all subsequent interactions.

**Transfer 5 — Aave withdrawal initiated**
```
Execution Contract → Null: 50,432,688.41618 aEthUSDT burned
```
Solver burns aEthUSDT to redeem underlying USDT from Aave reserve. 
Null address destination confirms permanent destruction of receipt 
token. Aave releases underlying only upon confirmed burn.

**Transfer 6 — USDT released**
```
Aave USDT Reserve → Execution Contract: 50,432,688.41618 USDT
```
Aave releases $50.4M USDT to solver. Aave interaction complete. 
This is the last moment the position is intact — $50.4M in a 
single wallet, ready to swap. Everything that follows destroys it.

**Transfer 7 — Uniswap output (optimistic transfer)**
```
Uniswap V3 USDT Pool → SushiSwap AAVE Pool: 17,957.81 WETH
```
Uniswap V3 sends swap output directly to SushiSwap pool — 
not back to solver. Solver pre-programmed direct pool-to-pool 
routing. Note: appears before Transfer 8 due to Uniswap V3's 
optimistic transfer pattern — output sent first, input collected 
via callback after.

**Transfer 8 — USDT into Uniswap**
```
Execution Contract → Uniswap V3 USDT Pool: 50,432,688.42 USDT
```
Solver sends $50.4M USDT into Uniswap V3 for first swap hop. 
Expected output at market rate: ~24,600 WETH. 
Actual output: 17,957 WETH. 
Loss at this step: ~$13.6M.

**Transfer 9 — SushiSwap output**
```
SushiSwap AAVE Pool → Execution Contract: 331.305 AAVE ($36,904)
```
17,957 WETH worth ~$38M entered SushiSwap AAVE pool. 
331 AAVE came out. $36.77M evaporated in a single pool 
interaction against $73,000 total liquidity. 
Pool completely drained of AAVE.

**Transfer 10 — AAVE deposited to Aave**
```
Execution Contract → Aave V3: 331.305 AAVE
```
Solver deposits recovered AAVE into Aave V3 to mint 
aEthAAVE receipt tokens. Final protocol interaction.

**Transfer 11 — aEthAAVE minted to Settlement**
```
Null → CoW Settlement: 331.305 aEthAAVE
```
Aave mints fresh aEthAAVE to CoW Settlement. 
Null origin confirms new mint. Settlement holds 
temporarily before delivery.

**Transfer 12 — Final delivery to victim**
```
CoW Settlement → Victim: 327.241 aEthAAVE ($36,389)
```
Victim receives 327.241 aEthAAVE — 4.064 less than minted 
in Transfer 11. The difference is solver surplus — CoW's 
compensation mechanism for the winning solver.

---

## The Math — Verified From Raw Log Data

![Image 20-03-26 at 7 43 PM](https://github.com/user-attachments/assets/25b5cd82-514e-4124-b076-16f3b02ee9c4)


**Source: Log 29 — SushiSwap AAVE Pool Sync Event**

Post-swap reserves:
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

Matches Transfer 9 exactly — 331.305 AAVE. 
Every last token drained from the pool. Mathematics confirmed 
from on-chain data without relying on any external source.

**Complete loss breakdown:**
```
Entry:    50,432,688 USDT        = $50,432,688
Uniswap:  17,957 WETH received   = $36,811,850  (-$13,620,838)
SushiSwap: 331 AAVE received     = $36,904      (-$36,774,946)
CoW fee:  4.064 aEthAAVE         = -$452
Final:    327.241 aEthAAVE       = $36,389
Total loss: $50,396,299 (99.93%)
```

---

## The Smoking Gun — Solver Had Full Information


![Image 20-03-26 at 7 39 PM](https://github.com/user-attachments/assets/f7768acd-fd1e-4885-b785-89d5e4f4fba6)

Internal transaction sequence immediately before SushiSwap execution:
```
staticcall — Get Reserves — Solver → SushiSwap AAVE Pool
staticcall — Balance Of — Solver → WETH
staticcall — Balance Of — Solver → WETH
call — Swap — Solver → SushiSwap AAVE Pool
```

The solver called GetReserves on the SushiSwap pool before routing. 
It retrieved the reserve data showing ~$73,000 total liquidity. 
It then executed the swap anyway.

This is not a case of missing information. The solver had the 
reserve data. Its algorithm failed to apply a minimum liquidity 
threshold check before routing. A basic validation — 
"if pool liquidity < order size × safety factor, reject this route" 
— would have prevented the entire loss.

---

## Accidental vs Deliberate — The Same Mechanic, Two Intents

This transaction structure is identical to a deliberate 
liquidity trap attack. Understanding the difference is 
critical for investigators.

**Accidental — what happened here:**
```
Old neglected pool (5+ years old)
No liquidity added before transaction
No liquidity removed after transaction  
Solver algorithm failure — routed with full reserve information
User ignored extraordinary slippage warning
```

**Deliberate liquidity trap — what it would look like:**
```
Attacker identifies intent-based DEX solver routing patterns
Attacker creates or selects tiny pool for target token
Attacker adds liquidity shortly before expected victim transaction
Attacker manipulates solver routing toward their pool
Victim executes — attacker's pool absorbs victim's funds
Attacker removes liquidity — extracts victim's funds plus own capital
```

**The on-chain signals are nearly identical:**
```
Both: large value in → tiny value out
Both: small pool selected for large order
Both: solver routed through low liquidity venue
```

**Five on-chain checks to distinguish accidental from deliberate:**

| Check | Accidental Signal | Deliberate Signal |
|-------|------------------|------------------|
| Pool age | Months/years old | Created days before attack |
| Pre-tx liquidity add | None | Mint event within 24-48hrs |
| Post-tx liquidity removal | None | Burn event within hours |
| Liquidity provider identity | Multiple LPs, old positions | Single fresh wallet |
| Solver wallet history | Legitimate bot, multiple users | Fresh wallet, single target |

**This case — all five checks confirm accidental:**
- Pool created 5 years 168 days ago ✅
- No Mint events before March 12 ✅
- No Burn events after March 12 ✅
- Multiple historical LPs ✅
- Solver has legitimate history across multiple users ✅

---

## Four Compliance Gaps in Intent-Based DEX Architecture

**Gap 1 — Attribution displacement:**

In a standard DEX swap the transaction initiator is the user. 
In CoW Protocol the on-chain transaction initiator is the solver bot. 
The victim's address only appears in internal logs — not as the 
from address on the transaction.

Automated tracing tools following the from address will trace 
the solver not the victim. Beneficial ownership requires 
manual reconstruction one layer deeper.

Sophisticated actors can exploit this — submit intents through 
CoW, let solvers execute, on-chain trail shows bot activity 
not human wallet. Obfuscation layer without using a mixer.

**Gap 2 — Solvers as unregulated intermediaries:**

Solver bots execute asset swaps, collect fees, and route 
funds across protocols with zero regulatory obligations. 
No KYC. No sanctions screening. No AML program.

A sanctioned entity can submit a swap intent. A solver 
executes it unknowingly and earns the surplus. The solver 
just processed a sanctioned transaction with no mechanism 
to prevent it.

Under VARA, MiCA, and FATF guidance — whether solvers 
constitute Virtual Asset Service Providers requiring 
registration is unresolved. The regulatory gap is live.

**Gap 3 — Slippage events as cover for deliberate extraction:**

As demonstrated in the comparative analysis above — 
accidental and deliberate liquidity trap transactions 
are structurally identical. Both produce the same 
catastrophic output. Both route through the same 
pool mechanics.

An investigator cannot distinguish them from token 
transfers alone. Manual reconstruction of pool history, 
liquidity provider identity, and solver wallet patterns 
is required. Automated tools will classify both as 
legitimate swap transactions.

**Gap 4 — Protocol fee capture without regulatory status:**


![Image 20-03-26 at 7 43 PM](https://github.com/user-attachments/assets/0caac5d2-cdf7-45e9-91e1-97849fd6dfa5)


CoW Protocol collected fees on a $50M transaction 
(Transfer 2 — hidden mint, Log 9 — feeAmount: 0 contradiction). 
No KYC on the user. No AML program. No regulatory registration.

Aave returned $600K in fees collected — acknowledging 
the ethical problem. But the legal obligation is unclear. 
Protocols facilitating nine-figure asset swaps and 
collecting fees without compliance infrastructure 
represent a systemic regulatory gap.

---

## Key Takeaways

**For investigators:**
This event is not detectable as suspicious from token 
transfers alone. The loss is only visible by reading 
the Uniswap Swap log price impact and applying constant 
product mathematics to the SushiSwap Sync log reserves. 
Manual transaction decoding is required — no analytics 
tool flagged this transaction as anomalous.

**For compliance professionals:**
Intent-based DEX architecture introduces attribution gaps, 
unregulated intermediaries, and identical transaction 
structures between user error and deliberate fraud. 
Current regulatory frameworks have not addressed any 
of these gaps.

**For protocol developers:**
The solver had full reserve information via GetReserves 
before routing. A minimum liquidity threshold check — 
rejecting any pool where liquidity depth is less than 
a defined percentage of order size — would have 
prevented this loss entirely.

---

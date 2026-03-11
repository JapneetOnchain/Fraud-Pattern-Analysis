# Fraud-Pattern-Analysis
On-chain fraud reconstruction from raw transaction data — rug pulls, exploits, and laundering patterns decoded without tools
# Fraud Pattern Analysis

On-chain fraud investigation casebook. Each case is reconstructed 
from raw transaction data — no blockchain analytics tools. 
Etherscan, manual tracing, creator wallet profiling.

Built while completing a self-directed crypto forensics curriculum
focused on exchange FIU and blockchain analytics firm workflows.

---

## Cases

| # | Fraud Type | Chain | Loss | Status |
|---|-----------|-------|------|--------|
| 001 | Soft Rug / LP Lock Theater | Ethereum | ~0.53 WETH + buyer losses | ✅ Complete |

---

## Methodology

Every case follows the same structure:

- **Mechanism** — how the fraud worked at transaction level
- **Red Flags** — signals visible before the rug executed
- **Timeline** — block-by-block reconstruction
- **Victim Impact** — who lost, how much, when
- **Investigative Next Steps** — what a real FIU analyst would do next

---

## Tools Used

Etherscan · Manual wallet tracing · No Chainalysis · No TRM



# Case 001 — GMPEPE Soft Rug / LP Lock Theater

**Date:** February 19–22, 2026  
**Chain:** Ethereum  
**Fraud Type:** Soft Rug Pull with LP Lock Theater  
**Creator Wallet:** 0x13D856e650B6a0770ae3ED3F6c8496A7a93424e8  
**Pool:** 0xff69B4a1638B605DfdD92B1C6a7e9e48E4A010cD  
**Loss:** 0.530 WETH extracted (includes funds deposited by buyers post-launch)

---

## Summary

Creator launched GMPEPE token with 0.5 WETH liquidity and immediately 
locked LP tokens on PinkSale for 3 days — a trust signal designed to 
attract buyers. During the lock window buyers purchased GMPEPE believing 
liquidity was protected. On February 22 the lock expired, LP tokens were 
reclaimed, and full liquidity was removed. All buyers were left holding 
worthless tokens against an empty pool.

---

## Mechanism

LP lock theater is a specific fraud pattern where a short lock duration 
creates false confidence while buyers accumulate. The lock is technically 
real — PinkSale genuinely holds the LP tokens until the specified date. 
The deception is the duration. 3 days is not a security measure. 
It is a marketing tool.

Legitimate lock durations: 6 months minimum. 1–2 years for credible projects.

---

## Transaction Sequence

**TX 1 — Pool Creation and Liquidity Add**  
`0x85f24d8e52788d0fa658c318a14b58f84a8e9ce773247264802e20ae520a3efa`  
Feb 19 2026 — Creator deposited 1,000,000,000 GMPEPE + 0.5 WETH.  
Received 0.707106 LP tokens. Pool created via OpenTrading() function  
in token contract — trading enabled immediately.

**TX 2 — LP Lock on PinkSale**  
Creator sent 0.707106 LP tokens to PinkSale time-lock contract.  
Lock expiry: February 22 2026. Advertised to buyers as security guarantee.

**TX 3 — LP Unlock**  
`0x7b7c0e2dc98396b159590e76e2127c9fb469daaabacd64e829f8bee2c6508ddf`  
Feb 22 2026 — PinkSale returned 0.707106 LP tokens to creator wallet  
upon lock expiry.

**TX 4 — Liquidity Removal (Rug Execution)**  
Feb 22 2026 — Creator burned LP tokens against pool.  
Withdrew 979,588,731 GMPEPE + 0.530 WETH.  
Pool emptied. Token price collapsed to zero.

---

## Red Flags Visible Before Rug Executed

**1 — MaxTxAmountUpdated event in launch transaction logs**  
Token contract imposed transaction size limits. Overwhelmingly  
associated with pump and dump tokens. Limits seller size  
while creator accumulates exit position.

**2 — 3-day lock duration**  
PinkSale lock until Feb 22 — 3 days from launch.  
Minimum credible lock is 6 months. 3 days signals intent to exit quickly.

**3 — Pool size: 0.5 WETH (~$1,000)**  
Trivially small liquidity. Any serious buyer moves price dramatically.  
Designed for pump not for legitimate trading infrastructure.

**4 — OpenTrading() function in token contract**  
Creator controlled exact moment trading began.  
Contracts with trading on/off switches often include  
additional hidden functions — honeypot mechanics,  
hidden mint capability, proxy drain functions.

---

## Timeline
```
Feb 19 08:31 UTC — Pool created, LP tokens locked 3 days
Feb 19–22       — Buyers purchase GMPEPE, pool accumulates WETH
Feb 22          — Lock expires, LP tokens returned to creator
Feb 22          — Liquidity removed, 0.530 WETH extracted
Feb 22          — Pool empty, GMPEPE worthless
```

---

## Victim Impact

Pool held 0.530 WETH at time of rug versus 0.500 WETH at launch.  
Delta of 0.030 WETH represents funds deposited by buyers  
during the 3-day lock window.

Every wallet that purchased GMPEPE between February 19–22  
holds tokens that are now worthless against an empty pool.

---

## Investigative Next Steps

1. Identify all wallets that purchased GMPEPE between Feb 19–22.  
Calculate individual losses and total victim pool.

2. Cross-reference creator wallet 0x13D856 against other token  
launches. Serial rug pullers reuse the same wallet or  
deploy from wallets funded by the same source.

3. Check PinkSale listing for GMPEPE — was the 3-day lock  
duration disclosed publicly or obscured in marketing materials?  
Misrepresentation of lock duration strengthens fraud case.

4. Profile 0x13D856 funding source. If funded from Tornado Cash  
or through a peel chain — creator anticipated law enforcement  
attention and pre-obfuscated identity.

---

## Key Takeaway

This fraud was invisible from the launch transaction alone.  
It was identified by following the creator wallet forward  
through subsequent transactions — standard FIU methodology.  
The transaction you are assigned is the thread. Pull it.

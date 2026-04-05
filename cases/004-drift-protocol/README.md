# Case 004: Drift Protocol : $285M Oracle Manipulation and Governance Takeover

**Chain:** Solana (cross-chain cashout to Ethereum)  
**Date:** April 1, 2026  
**Amount:** $285.26M  
**Classification:** Oracle Manipulation + Governance Takeover + Privileged Access Abuse  
**DPRK Attribution:** Flagged by multiple blockchain analytics firms  
**Investigation Date:** April 4, 2026  

---

## Overview

On April 1, 2026, Drift Protocol, one of Solana's largest perpetual DEXs with over $550M in TVL, was drained of $285M in under 12 minutes. This was not a smart contract bug. The attacker spent 20 days building fake token infrastructure, compromising governance keys, and preparing pre-signed transactions. When everything was ready, two governance approvals submitted one second apart handed full admin control to the attacker. The drain followed immediately.

This case documents the full attack chain from primary on-chain sources. Every finding listed here was verified independently via Solscan, Arkham Intelligence, and Etherscan. Secondary sources are cited where used.

---

## Attack Architecture

The exploit required three components operating in sequence. Any single component failing would have stopped the attack.

| Component | What It Did | When |
|-----------|-------------|------|
| Fake token infrastructure | Created CVT token, Raydium pool, and Switchboard oracle | March 12, 2026 |
| Governance compromise | Pre-signed transactions from compromised multisig keys | March 23 to March 31 |
| Execution | Admin takeover, CVT market creation, vault drain | April 1, 2026 |

---

## Layer 1: Weapon Construction (March 12, 2026)

### CVT Token Creation

At 00:58:29 UTC on March 12, wallet `FnYXwy7qEtGV4cj1Sf6tht7VDsiytFjJeF9yd4LpAjjx` minted 750 million CarbonVote Token (CVT) in a single transaction. All tokens went to the creator's own wallet. The token had 36 total holders as of the investigation date, consistent with a purpose-built attack token never distributed publicly.

**TX:** `2UnWSA4VnAjoPVUzg9VRKcVhDNwKq1LwZUGX3A6rrEPewrPksbUvgbH3AdzsHSwLPiJJcbns1Kb9hGABKhW8tMfK`

![8D13CAF2-F0BB-4D2D-88BA-764B66CB8702](https://github.com/user-attachments/assets/f2636630-fa9a-4055-b25d-4da53bf1362e)

---

### Raydium Pool Creation and $500 Price Seeding

At 01:03:54 UTC, five minutes after minting CVT, the creator opened a USDC/CVT liquidity pool on Raydium V4 and seeded it with exactly 500 USDC and 500 CVT. Depositing equal amounts established a starting price of $1 per CVT.

This $500 deposit created the pricing foundation for $750 million in fake collateral. The ratio of deposit to apparent collateral value was 1 to 1,500,000.

**TX:** `3KbWCN2Leska9hLfdNMjBTwuFFJ9TeAoupPW57NfEpQ5pmykc2RZnwkp2tNuxSPza3SMdHfUiARrkn1CQHeotxbc`

![F8486594-0EB9-49D0-98B7-BE86703F8EC9](https://github.com/user-attachments/assets/23f217a9-9809-4671-8893-4c7100fe7634)

---

### Switchboard Oracle Creation

At 01:07:54 UTC, the creator initialized a Switchboard On Demand oracle pointed at the Raydium USDC/CVT pool. The creator wallet was set as the oracle authority. This oracle would later be forced onto Drift Protocol as the official price feed for CVT.

**TX:** `ekyEfoeqpEnzVYVSUmqGGN7B34ALBKsVsHubDrDKgwzPujcSxQrRvrMwz9W5RAc9GwQEWyCTCSCUSaM7w2ScQp5`  
**Oracle address:** `B6X3YojuJcfHnTTre68b2XxDPCVTfeA4N7ZJEpME2nFp`  
**Authority:** `FnYXwy7qEtGV4cj1Sf6tht7VDsiytFjJeF9yd4LpAjjx` (CVT creator)

![E7B4496E-F0BC-4EE0-A5B8-4EC19FBB8A11](https://github.com/user-attachments/assets/8e628db9-d195-42e8-b6da-c9bcbb50af68)
![8A5CBD81-7D11-441B-A8B9-83F2BEEF10A2](https://github.com/user-attachments/assets/aa3d6d6d-d2e3-4da5-8b40-06aa7bc73e43)

**Key finding:** This oracle address `B6X3YojuJcfHnTTre68b2XxDPCVTfeA4N7ZJEpME2nFp` was confirmed to be the same address used in the `initializeSpotMarket` instruction on April 1. The attacker created the oracle on March 12 and used it 20 days later. Both ends of this chain were verified from primary on-chain data.

---

## Layer 2: Governance Compromise

### How Durable Nonces Work

Solana transactions normally expire within 60 seconds because they reference a recent blockhash. Durable nonces bypass this by substituting a stable nonce account value. A transaction using a durable nonce stays valid indefinitely. This allowed the attacker to collect signed transactions from compromised keyholders weeks in advance and submit them at a precise moment.

---

### Attack Preparation: Funding the Attacker Infrastructure

On March 31, the CVT creator wallet funded the attacker admin wallet with 3 SOL. This was the first ever transaction on that wallet, confirming it was created specifically for this attack.

**TX:** `iYoYSB8BFCne3SX4uBiiEE2658nteiRkwGa3W8mo977XoLQYabFGFfZVWFYqA2NuyK9pWekHsyQJx9u7SJmBHVy`  
**From:** `FnYXwy7qEtGV4cj1Sf6tht7VDsiytFjJeF9yd4LpAjjx` (CVT creator)  
**To:** `H7PiGqqUaanBovwKgEtreJbKmQe6dbq6VTrw6guy7ZgL` (Drift Compromised Admin Signer)  
**Amount:** 3 SOL

![7C979FC1-74AB-4E39-B261-7C19BCB6638A](https://github.com/user-attachments/assets/7516604d-dcc2-41cb-a795-29d528ab380a)


---

### Original Finding: CVT Creator Funded the Multisig Signer Directly

This finding was not present in any published report reviewed during this investigation.

At 16:02:38 on April 1, the CVT creator wallet sent 0.01 SOL to `6UJbu9ut5VAsFYQFgPEa5xPfoyF5bB5oi4EknFPvu924`, labeled Drift Multisig #3 by Solscan. This transfer happened 2 minutes and 41 seconds before the governance execution at 16:05:19. It was confirmed from both the sender and receiver transaction histories.

**TX:** `2Q9qXdCbqC5bow9t96833nrC6mb9LrNzcFZtKMofdkqigWWA6yUXGCJphCMPb8Kvaxy3A8ZD1TyGDeAcEiVfB8uR`  
**From:** `FnYXwy7qEtGV4cj1Sf6tht7VDsiytFjJeF9yd4LpAjjx`  
**To:** `6UJbu9ut5VAsFYQFgPEa5xPfoyF5bB5oi4EknFPvu924` (Drift Multisig #3)  
**Time:** 16:02:38 Apr 01, 2026

![767928BF-4CF7-4721-8EED-B53C4701A657](https://github.com/user-attachments/assets/0711544d-947f-4219-b415-d3ffa773825d)


This directly connects the CVT token creator to the governance attack. The same wallet that built the fake token infrastructure also funded the compromised multisig signer minutes before the attack execution.

---

### The One-Second Governance Execution


![D087D638-DBE4-4120-9F7A-41947970568F](https://github.com/user-attachments/assets/4a6c1061-70b4-406e-8a12-5b02587a50fb)


At 16:05:18, Drift Multisig #2 submitted a proposal to transfer admin control. One second later at 16:05:19, Drift Multisig #3 approved and executed it in a single atomic transaction.

16:05:18 — TX 2HvMSgDEfKh... — vaultTransactionCreate — Drift Multisig #2

16:05:19 — TX 4BKBmAJn6Tds... — proposalApprove + vaultTransactionExecute — Drift Multisig #3

Gap: 1 second

In legitimate 2/5 multisig governance, two independent humans review and sign separately. A one-second gap between proposal creation and full execution by two different signers indicates the same entity controlled both keys simultaneously.

Transaction `4BKBmAJn6Tds...` contained three instructions in one atomic operation:

1. AdvanceNonce (confirms durable nonce pre-signing)
2. ProposalApprove (second multisig signature)
3. VaultTransactionExecute calling Drift V2 Program UpdateAdmin

![0DED1F3F-40CA-4FA0-8891-F13D03F409BC](https://github.com/user-attachments/assets/b5e7622e-b2d6-4d57-a2c3-a1c8c637b759)


The Drift protocol program log from this transaction recorded the admin change on-chain:

Program log: Instruction: UpdateAdmin
Program log: admin: AiLGdNitMjv8n5HMS7HAdV2kaeJZZFd4jdfn5xp1PKrW
-> H7PiGqqUaanBovwKgEtreJbKmQe6dbq6VTrw6guy7ZgL

This is Drift's own protocol writing to the blockchain. The legitimate admin was replaced by the attacker's wallet. This is irrefutable primary source evidence.

---

## Layer 3: Attack Execution (April 1, 2026)

### CVT Spot Market Initialization

At 16:05:39, 20 seconds after receiving admin control, the attacker created a CVT spot market inside Drift. The raw instruction data from this transaction confirms:

- Oracle source: `switchboardOnDemand`
- Oracle account: `B6X3YojuJcfHnTTre68b2XxDPCVTfeA4N7ZJEpME2nFp` (the same oracle created by FnYXwy on March 12)
- Asset tier: `collateral`
- Initial asset weight: `10000` (100%, no haircut applied)
- Spot market number: 63

**TX:** `4a5962Rdqd9pkXtk9DMQ9ZYhdGb2k9gPw71GvukJgELhxbCY5gm1c1hhKdwuGefyqJ3XMvihUTDNDn3qbXnst82X`

![1FA45450-5242-436E-807D-24DB1DB010CD](https://github.com/user-attachments/assets/a9603b86-c097-40ed-b57e-24fb86320060)

![72AA9F16-26C5-4BB3-94EC-62792D22BC17](https://github.com/user-attachments/assets/9b0ccbdb-58da-4fc7-8120-84ebc5c218e3)


---

### Withdrawal Limit Removal

In the same transaction, the attacker raised withdrawal guard thresholds across multiple Drift markets simultaneously. The program logs recorded the exact before and after values:

![A9B7CE94-AA11-4AE9-AA7B-3E84A14981FF](https://github.com/user-attachments/assets/477d53b6-70de-4cf3-8546-588b7b41d28f)


| Market | Asset | Before | After | Multiplier |
|--------|-------|--------|-------|------------|
| 0 | USDC | 25,000,000,000,000 | 500,000,000,000,000 | 20x |
| 4 | WETH | 200,000,000,000 | 500,000,000,000,000 | 2,500x |
| 17 | dSOL | 5,000,000,000,000 | 500,000,000,000,000 | 100x |
| 19 | (unnamed) | 5,000,000,000 | 500,000,000,000,000 | 100,000x |
| 27 | (unnamed) | 10,000,000,000 | 500,000,000,000,000 | 50,000x |

Safety caps designed to limit withdrawal velocity were effectively removed. The drain then proceeded through 33 borrow-loop transactions over the next 12 minutes.

---

### Assets Drained

| Token | Amount | USD Value |
|-------|--------|-----------|
| JLP | 42.72M | $159.35M |
| USDC | 71.42M | $71.42M |
| cbBTC | 164.35 | $11.29M |
| USDT | 5.65M | $5.65M |
| USDS | 5.25M | $5.25M |
| WETH | 2,200.59 | $4.69M |
| dSOL | 45,292.21 | $4.47M |
| WBTC | 63.47 | $4.36M |
| Fartcoin | 23.37M | $4.11M |
| JitoSOL | 33,976.51 | $3.60M |
| syrupUSDC | 2.87M | $3.32M |
| INF | 21,241.62 | $2.50M |
| mSOL | 17,418.92 | $1.99M |
| bSOL | 9,474.33 | $1.02M |
| EURC | 583,980.69 | $677K |
| zBTC | 8.61 | $587K |
| USDY | 477,375.42 | $539K |
| JUP | 2.62M | $431K |
| **Total** | | **$285.26M** |

---

## Fund Flow

### Solana Dispersal

All stolen assets were collected at `HkGz4KmoZ7Zmk7HN6ndJ31UJ1qZ2qgwQxgVqQwovpZES`, labeled Drift Exploiter 1 by Solscan. This wallet was funded by the CVT creator wallet on March 24, confirming the direct link between token creation and fund collection.

From the primary drain wallet, funds were dispersed to a network of at least 12 attacker-controlled wallets, all labeled Drift Protocol Exploiter by Arkham Intelligence. Stolen assets were converted to USDC, SOL, and wrapped assets via Jupiter swaps during the drain.

---

### Cross-Chain Bridge to Ethereum

The primary bridge used was Circle's Cross-Chain Transfer Protocol (CCTP). A CCTP burn transaction was verified on Solana:

**TX:** `3WgPgTyQzDugZCUTtyYN9usyyd81nnmT2eZBHWfG3FEWu4jfNaY9j6RgB1w48TvNfghB59cxmvcUDVMD1EX2QQ1V`  
**Time:** 17:51:06 Apr 01, 2026  
**Program:** Circle CCTP Token Messenger Minter V2  
**Action:** BURN 743,731 USDC  
**Signer:** Drift Protocol Exploiter (7RoMq) — Arkham label

![F8A1E4AE-02F4-41A7-B2D1-097180998BDA](https://github.com/user-attachments/assets/ecac14c6-7b5c-4129-8f19-154a0670ed95)

![F07F48C3-BE35-4654-9AE5-056BD29F1CE8](https://github.com/user-attachments/assets/ce1dfe87-730c-4611-9867-c10edce7fddd)

A parallel Chainflip bridge transaction moved SOL directly to ETH, arriving at Ethereum address `0xd91a122b585bc588c9a48d0995ee0d7b4f8ab7dd`, which then forwarded 787.97 ETH to the primary Ethereum exploiter wallet.

![20816843-A32B-436D-B9D0-F6D11FB25707](https://github.com/user-attachments/assets/8ba6a450-3e84-47c7-9bf2-cf6b87ab698c)

On Ethereum, USDC arrivals were converted to ETH via CoW Protocol Settlement in real time. The total ETH accumulated across the entity reached 129,066 ETH as reported by on-chain investigators on April 1.

---

### Current Status: April 4, 2026 (Original Finding)

The primary Ethereum exploiter wallet `0xD3FEEd5DA83D8e8c449d6CB96ff1eb06ED1cF6C7`, labeled Drift Exploiter 1 by Etherscan, was checked directly on the investigation date:

ETH Balance: 24,881.975878091003662262 ETH
USD Value:   $50,748,741.43 (@ $2,039.58/ETH)
Status:      No outgoing transactions

![2F907251-9CD0-4645-ABAE-C74159FE3ADA](https://github.com/user-attachments/assets/abac95c1-50d1-4589-b92a-f6de8dc0e3d8)


The reduction from 129,066 ETH on April 1 to 24,881 ETH on April 4 indicates approximately 104,000 ETH has dispersed to secondary addresses. $50M in the primary wallet remains identifiable and unlaundered as of this investigation.

---

## Compliance Findings

### USDY Freeze by Ondo Finance

Among the assets drained was 477,375 USDY, a yield-bearing stablecoin issued by Ondo Finance backed by US Treasury bills. During this investigation, the USDY was identified still sitting in the primary Solana drain wallet. Ondo Finance subsequently froze these tokens, preventing the attacker from accessing them.

This demonstrates a compliance-specific insight that security-focused reports often miss. When stolen assets include tokens issued by regulated entities, those issuers retain the ability to intervene through freezes. Regulated asset issuers embedded within DeFi protocols can act as emergency circuit breakers.

---

### Circle CCTP as a Compliance Chokepoint

The use of Circle's CCTP as the primary bridge created a traceable, regulated chokepoint. Circle as a regulated entity is subject to law enforcement requests and OFAC compliance obligations. Every CCTP burn on Solana and the corresponding mint on Ethereum is permanently recorded and linked. This is materially different from fully decentralized bridges.

---

### DPRK Attribution and OFAC Implications

Multiple blockchain analytics firms flagged this exploit as consistent with DPRK Lazarus Group methodology. This was described as the 18th suspected DPRK attack of 2026. DPRK attribution carries direct OFAC implications for any VASP or financial institution that processes funds tracing back to these wallets, even unknowingly.

---

## Key Addresses

### Solana

| Label | Address |
|-------|---------|
| CVT Creator / Attacker Funder | `FnYXwy7qEtGV4cj1Sf6tht7VDsiytFjJeF9yd4LpAjjx` |
| CVT Token | `G84LEhbNMR1yYbHgHbnNYNSK8mpTKcazh5jcW5yMPQKo` |
| Switchboard Oracle | `B6X3YojuJcfHnTTre68b2XxDPCVTfeA4N7ZJEpME2nFp` |
| Attacker Admin (Compromised) | `H7PiGqqUaanBovwKgEtreJbKmQe6dbq6VTrw6guy7ZgL` |
| Drift Multisig #2 (Compromised) | `39JyWrdbVdRqjzw9yyEjxNtTbTKcTPLdtdCgbz7C7Aq8` |
| Drift Multisig #3 (Compromised) | `6UJbu9ut5VAsFYQFgPEa5xPfoyF5bB5oi4EknFPvu924` |
| Primary Drain Wallet | `HkGz4KmoZ7Zmk7HN6ndJ31UJ1qZ2qgwQxgVqQwovpZES` |
| Drift Vault | `JCNCMFXo5M5qwUPg2Utu1u6YWp3MbygxqBsBeXXJfrw` |
| Drift V2 Program | `dRiftyHA39MWEi3m9aunc5MzRF1JYuBsbn6VPcn33UH` |

### Ethereum

| Label | Address | Status (Apr 4) |
|-------|---------|----------------|
| Drift Exploiter 1 (primary) | `0xD3FEEd5DA83D8e8c449d6CB96ff1eb06ED1cF6C7` | 24,881 ETH unspent |
| Exploiter EOA 2 | `0xAa843eD65C1f061F111B5289169731351c5e57C1` | |
| Exploiter EOA 3 | `0xbDdAE987FEe930910fCC5aa403D5688fB440561B` | |
| Exploiter EOA 4 | `0x0FE3b6908318B1F630daa5B31B49a15fC5F6B674` | |
| Chainflip Intermediate | `0xd91a122b585bc588c9a48d0995ee0d7b4f8ab7dd` | Emptied |

---

## Key Transactions

| Event | Transaction Hash |
|-------|-----------------|
| CVT Token Mint | `2UnWSA4VnAjoPVUzg9VRKcVhDNwKq1LwZUGX3A6rrEPewrPksbUvgbH3AdzsHSwLPiJJcbns1Kb9hGABKhW8tMfK` |
| Raydium Pool Creation | `3KbWCN2Leska9hLfdNMjBTwuFFJ9TeAoupPW57NfEpQ5pmykc2RZnwkp2tNuxSPza3SMdHfUiARrkn1CQHeotxbc` |
| Oracle Creation | `ekyEfoeqpEnzVYVSUmqGGN7B34ALBKsVsHubDrDKgwzPujcSxQrRvrMwz9W5RAc9GwQEWyCTCSCUSaM7w2ScQp5` |
| CVT Creator funds H7PiGq | `iYoYSB8BFCne3SX4uBiiEE2658nteiRkwGa3W8mo977XoLQYabFGFfZVWFYqA2NuyK9pWekHsyQJx9u7SJmBHVy` |
| CVT Creator funds Multisig #3 | `2Q9qXdCbqC5bow9t96833nrC6mb9LrNzcFZtKMofdkqigWWA6yUXGCJphCMPb8Kvaxy3A8ZD1TyGDeAcEiVfB8uR` |
| Admin Transfer Proposal | `2HvMSgDEfKhNryYZKhjowrBY55rUx5MWtcWkG9hqxZCFBaTiahPwfynP1dxBSRk9s5UTVc8LFeS4Btvkm9pc2C4H` |
| Admin Transfer Execution | `4BKBmAJn6TdsENij7CsVbyMVLJU1tX27nfrMM1zgKv1bs2KJy6Am2NqdA3nJm4g9C6eC64UAf5sNs974ygB9RsN1` |
| CVT Market Initialization | `4a5962Rdqd9pkXtk9DMQ9ZYhdGb2k9gPw71GvukJgELhxbCY5gm1c1hhKdwuGefyqJ3XMvihUTDNDn3qbXnst82X` |
| CCTP Bridge Burn | `3WgPgTyQzDugZCUTtyYN9usyyd81nnmT2eZBHWfG3FEWu4jfNaY9j6RgB1w48TvNfghB59cxmvcUDVMD1EX2QQ1V` |

---

## Original Findings Summary

The following findings were not present in any published report reviewed during this investigation:

1. The CVT creator wallet directly funded Drift Multisig #3 at 16:02:38 on April 1, two minutes and 41 seconds before the governance execution. Verified from both wallet transaction histories.

2. The attacker admin wallet H7PiGq had zero transaction history before March 31. It was a brand new wallet created specifically for this attack.

3. Oracle address B6X3Yoju was traced from creation by FnYXwy on March 12 to its use inside the initializeSpotMarket instruction on April 1. The same attacker-created oracle address appears in both events.

4. The Drift program log from transaction 4BKBmAJn6T shows the exact admin change on-chain: `admin: AiLGdN... -> H7PiGq...`

5. The exact withdrawGuardThreshold before and after values for each affected market were extracted from program logs.

6. As of April 4, 2026, 24,881 ETH ($51M) remains in the primary Ethereum exploiter wallet, unlaundered. This is current data not available in any prior report.

7. 477,375 USDY was identified in the Solana drain wallet as a regulated asset with freeze capability. Ondo Finance subsequently froze these tokens.

---

*Investigation conducted April 4, 2026. All Solana findings verified via Solscan primary source data. Ethereum findings verified via Etherscan. Cross-chain flow analysis via Arkham Intelligence.*

# Fraud Pattern Analysis

On chain investigation casebook. Fraud reconstruction, protocol 
failures, and attack pattern analysis decoded from raw transaction 
data without tools.

Each case is reconstructed from raw Etherscan data with 
no Chainalysis, no TRM Labs, no blockchain analytics tools.

Built during a self-directed crypto forensics curriculum focused 
on exchange FIU and blockchain analytics firm workflows.

---

## Cases

| # | Case | Chain | Loss | Status |
|---|------|-------|------|--------|
| [001](cases/001-gmpepe-soft-rug/README.md) | Soft Rug / LP Lock Theater - GMPEPE | Ethereum | ~0.53 WETH + buyer losses | ✅ Complete |
| [002](cases/002-cow-protocol-solver-failure/README.md) | CoW Protocol Solver Routing Failure $50M Slippage Event | Ethereum | $50,395,611 | ✅ Complete |
| [003](cases/003-resolv-usr-exploit/README.md) | Privileged Key Abuse - Resolv USR Exploit | Ethereum | ~$25M | ✅ Complete |

---

## Methodology

Every case follows the same structure:

**Mechanism** -how the fraud or failure worked at transaction level

**Evidence** -raw Etherscan data, logs, internal transactions

**Mathematics** -on-chain verified calculations where applicable

**Investigative Framework** -how to distinguish this pattern 
from similar legitimate activity

**Compliance Gaps** -regulatory blind spots exposed by the case

---

## Tools Used

Etherscan · Manual wallet tracing · No Chainalysis · No TRM

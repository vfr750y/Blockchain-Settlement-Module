# High-Level Design Document: Blockchain Settlement Module (BSM) v5.0 (ZKsync Integration)

**To:** Executive Steering Committee, CISO, Lead Systems Architect

**From:** Solutions Architecture Office

**Date:** January 23, 2026

**Status:** Final Draft - ZKsync Fast-Settlement Architecture

---

## 1. Executive Summary

The **BSM v5.0** is a critical upgrade to the bank’s blockchain infrastructure, transitioning from Ethereum Layer 1 (L1) to **ZKsync Era (Layer 2)**. This shift addresses the primary bottleneck identified in previous versions: the "Settlement Gap" caused by Ethereum's 12.8-minute finality window.

By leveraging ZKsync’s **Zero-Knowledge Rollup** technology and **Native Account Abstraction**, the bank can now offer near-instant transaction confirmation ("Soft Finality") while maintaining the "Zero-Access" security posture. This version continues the **Self-Funded model**, where users pay for their own transaction fees (gas) via their L2-based Smart Contract Wallets (SCW).

---

## 2. Updated Logical Architecture

The architecture moves away from the external ERC-4337 EntryPoint dependency on L1 and integrates directly with the ZKsync **Bootloader** and **Sequencer**.

### BSM v5.0 Component Overview

| Component | Function | Status |
| --- | --- | --- |
| **Mobile Banking App** | User interface for initiating transfers and signing transactions. | Enhanced (L2 Support) |
| **Secure Enclave (TEE)** | Stores the private key on the user's device for biometric signing. | Unchanged |
| **BSM Gateway** | The bank's internal API that authenticates users and prepares L2 transactions. | Updated (ZKsync SDK) |
| **Sentinel Mesh** | Orchestrator that submits transactions to the ZKsync Sequencer. | **New (L2 Integration)** |
| **ZKsync Sequencer** | The off-chain operator that orders and executes transactions instantly. | **New (Replaces L1 Mempool)** |
| **ZKsync Bootloader** | System contract on L2 that handles native Account Abstraction (AA) logic. | **New (Replaces L1 EntryPoint)** |
| **Sentinel Warden** | Monitors the L2 state for "Soft" and "Hard" (L1-verified) finality. | Updated (L2 Monitoring) |

---

## 3. The "New" Settlement Gap: L1 vs. ZKsync

The transition to ZKsync effectively eliminates the 12.8-minute "Justified" state uncertainty for retail users.

| Finality Stage | Ethereum L1 (Old) | ZKsync Era (New) | Bank Action |
| --- | --- | --- | --- |
| **Inclusion / Soft Finality** | ~12 Seconds | **~1 - 2 Seconds** | **Update UI:** Transaction "Pending" |
| **Economic Certainty** | 12.8 Minutes | **~10 - 20 Seconds** | **Internal Credit:** Funds usable in-app |
| **Hard Finality (L1 Verified)** | 12.8 Minutes | ~15 - 60 Minutes* | **Settlement Complete:** Permanent Ledger Update |

**Note: While ZKsync takes longer to reach full L1 verification, the "Soft Finality" provided by the Sequencer is cryptographically backed by the validity proof that will inevitably follow, reducing rejection risk to near-zero.*

---

## 4. BSM v5.0 Data Flow (Mermaid)

This diagram illustrates the streamlined flow using ZKsync's native AA and instant sequencer.

```mermaid
graph TD
    subgraph User_Zone [User Device]
        App[Mobile Banking App]
        TEE[Secure Enclave / TEE]
    end

    subgraph Bank_Internal [Bank Internal Network - Secure Zone]
        GW[BSM Gateway]
        Mesh[Sentinel Mesh <br/>'L2 Orchestrator']
        Warden[Sentinel Warden <br/>'L2 Monitor']
    end

    subgraph ZKsync_L2 [ZKsync Era Network]
        Seq[ZKsync Sequencer]
        BL[Bootloader <br/>'Native AA']
        SCW[User Smart Account]
    end

    subgraph Ethereum_L1 [Ethereum Mainnet]
        Verify[ZK Verifier Contract]
    end

    %% Flow Steps
    App -- 1. Request Transaction --> GW
    GW -- 2. Create L2 Transaction --> Mesh
    Mesh -- 3. Push to Device --> App
    App -- 4. Biometric Sign --> TEE
    TEE -- 5. Signed Tx --> App
    App -- 6. Broadcast to Mesh --> Mesh
    Mesh -- 7. Submit to Sequencer --> Seq
    Seq -- 8. Instant Soft Finality --> BL
    BL -- 9. Execute Transfer --> SCW
    Seq -- 10. Confirmation --> Mesh
    Mesh -- 11. Success Alert --> App
    Seq -. 12. Generate Validity Proof .-> Verify
    Verify -. 13. Hard Finality Alert .-> Warden
    Warden -- 14. Close Ledger Entry --> GW

```

---

## 5. Detailed Component Breakdown

### 5.1 ZKsync Sequencer (The "Fast Lane")

In BSM v4.0, we broadcast to the public Ethereum mempool and waited. In v5.0, the **Sentinel Mesh** connects directly to the ZKsync Sequencer.

* **How it works:** The Sequencer receives the user's transaction, confirms the gas fee (paid by the user's SCW), and provides an immediate receipt.
* **Benefit:** This reduces the "Pending" state from 12 seconds to sub-second, allowing the Banking App to feel as responsive as a traditional database update.

### 5.2 ZKsync Bootloader & Native AA

We have removed the L1 ERC-4337 EntryPoint. ZKsync utilizes a **Native Account Abstraction** model.

* **How it works:** Every account on ZKsync is essentially a smart contract. The **Bootloader** is a special system contract that acts as the entry point for all transactions. It handles the validation of the user's biometric signature and ensures the SCW has enough ETH for gas.
* **Benefit:** Native AA is more gas-efficient than the ERC-4337 overlay on L1, further reducing costs for the customer.

### 5.3 Sentinel Warden (The Auditor)

The Warden’s role is simplified but expanded.

* **How it works:** It monitors the ZKsync API for the `finalized` status of a batch.
* **States Tracked:**
1. **Committed:** The transaction is included in a batch and sent to Ethereum L1.
2. **Proven:** The ZK-proof has been generated and verified by the L1 Verifier contract.


* **Benefit:** Once a transaction is "Proven," it is mathematically impossible to reverse, providing a higher level of security than L1's "Economic Finality."

---

## 6. Security & Compliance Considerations

1. **Zero-Access Key Management:** The move to L2 does not change the fact that the bank never sees the private key. Signing remains localized in the user's mobile TEE.
2. **ZK-Privacy for Compliance:** While the design uses a public ZKsync instance, future iterations can utilize ZKsync's **Prividium** (private ZK chains) to settle sensitive inter-bank transactions without exposing PII (Personally Identifiable Information) on the public ledger.
3. **AML/KYC Integration:** The BSM Gateway continues to perform off-chain KYC checks before the Sentinel Mesh allows a transaction to be submitted to the Sequencer.

---

## 7. Glossary

* **ZK-Rollup:** A layer 2 scaling solution that uses validity proofs to scale Ethereum.
* **Soft Finality:** A promise by the L2 sequencer that a transaction will be included in the next batch.
* **Hard Finality:** The state where a transaction's validity proof is accepted by the Ethereum L1 contract.
* **Validity Proof (SNARK):** A mathematical proof that a batch of transactions is valid, requiring no "challenge period."
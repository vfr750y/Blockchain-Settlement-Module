## Cross ledger settlement Gap Analysis

<div style="text-align: center;">
<img src="./TheSettlementGap.png" alt="The Settlement Gap" width="250" height="250">
</div>

**How do we allow customers to with the Ethereum blockchain?**

### Basic Problem Definition

Using the current digital experience, i.e. the online banking app, how can we allow customers to buy and sell cryptocurrency without exposing the bank or customer to risk? The main problems are a delayed transaction finality and a variable transaction cost. 
These problems are significant but not insurmountable.

**Delayed Transaction Finality**
Your bank’s traditional ledger and the Ethereum blockchain differ when it comes to settlement time and transaction certainty.
In a traditional bank, when you move money from savings to checking, the bank’s computer simply updates a database. 
Once the screen says "Success," it is deterministic—meaning it is a done deal.

When interacting with the Ethereum mainnet, the consensus layer (the part of Ethereum that validates the next block in the chain), is, for a time, probabilistic in nature. There are three distinct stages for a block first broadcast, Pending, Justified and Finalized.
This table describes those states:

| State | Age of Transaction | Rejection Risk | Bank Action |
| :--- | :--- | :--- | :--- |
| **Pending** | 0 - 12 Seconds | **High** | Ignore (Awaiting Block Inclusion) |
| **Justified** | ~6.4 Minutes | **Extremely Low** | Prepare Ledger (Internal Hold Created) |
| **Finalized** | **12.8 Minutes** | **Zero (Economic)** | **Settlement Complete** (Credit Bank Balance) |

The problem is this 12 minute window when the transaction is unconfirmed in Ethereum. As a bank, there can be no doubt about the transaction's state at any time.

**Variable Transaction Cost**
Each Ethereum transaction requires gas, a minimum charge plus a variable charge which together make the transaction fee. The higher the fee, the shorter the time to finality. If the bank does not want to pay up-front for the user's transaction, it needs to ensure the user has enough currency to pay the transaction fee.

### Other considerations.
- A bank should not have access to a customer's private key or mnemonic phrase.
- All communications should be encrypted and restricted through firewalls and APIs.
- AML compliance needs to be in place to restrict money laundering through the blockchain.






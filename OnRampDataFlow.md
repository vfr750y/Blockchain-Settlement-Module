```mermaid
graph TD
    %% Define Actors and Components
    Customer([Customer via Mobile App])

    subgraph "Bank In-House Infrastructure"
        OM[Orchestration Middleware / API Gateway]
        CBS[(Core Banking System - Fiat Ledger)]
        SL[(Shadow / Sub-Ledger - Crypto Balances)]
        Recon[Reconciliation Engine]
    end

    subgraph "External Composable Integrations"
        KYC["Identity & Compliance Service (KYC/KYT)"]
        LP[Liquidity Provider / Exchange]
        Custody[Digital Asset Custody Provider]
        Rails["Payment Rails (RTP/FedNow)"]
    end

    %% --- FLOW INITIATION ---
    Customer -- "1. Request 'Buy $100 BTC'" --> OM

    %% --- VALIDATION PHASE ---
    OM -- "2. Check Fiat Balance" --> CBS
    CBS -- "Funds OK" --> OM
    OM -- "3. Verify Transaction Eligibility" --> KYC
    KYC -- "Approved" --> OM

    %% --- QUOTING PHASE ---
    OM -- "4. Request Live BTC Quote" --> LP
    LP -- "Quote Received" --> OM
    OM -- "5. Present Quote & Await Confirmation" --> Customer

    %% --- EXECUTION PHASE (If Confirmed) ---
    Customer -- "6. Confirm Transaction" --> OM

    %% Fiat Leg
    OM -- "7a. Debit Customer Fiat Account" --> CBS
    OM -- "7b. Initiate Fiat Settlement to LP" --> Rails
    Rails -.-> LP

    %% Crypto Leg
    OM -- "7c. Execute Buy Order against Quote" --> LP
    LP -- "8. Deliver BTC directly to Vault" --> Custody
    Custody -- "9. Confirm BTC Receipt & New Balance" --> OM

    %% --- RECORDING & COMPLETION ---
    OM -- "10. Update Customer Crypto Balance" --> SL
    OM -- "11. Notify Success & Show New Balance" --> Customer

    %% --- ASYNC POST-PROCESS ---
    CBS -. "12. Daily Data Feed" .-> Recon
    SL -. "Crypto Data Feed" .-> Recon
    Custody -. "Vault Data Feed" .-> Recon

    %% Styling
    classDef internal fill:#e1f5fe,stroke:#01579b,stroke-width:2px,color:#000;
    classDef external fill:#fff3e0,stroke:#e65100,stroke-width:2px,stroke-dasharray: 5 5,color:#000;
    classDef actor fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px,color:#000;

    class OM,CBS,SL,Recon internal;
    class KYC,LP,Custody,Rails external;
    class Customer actor;```
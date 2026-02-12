# Phase 2: System Design and Architecture

This chapter details the design and architectural blueprint for the proposed Blockchain-Based Land Ownership Verification System. The design is meticulously aligned with the project's objectives, the research gap identified in Phase 1, and the technical constraints of a low-resource environment typical of developing nations like Nigeria.

## 1. System Overview

The proposed system, tentatively named **SecureLand Registry (SLR)**, is a hybrid decentralized application (DApp) designed to enhance the transparency, security, and efficiency of land administration. It operates as a prototype to demonstrate the feasibility of integrating blockchain technology into a traditionally centralized and paper-based land registry system.

### 1.1 Justification for Blockchain Adoption

The decision to utilize blockchain technology, specifically a public permissioned or consortium chain model simulated on an Ethereum testnet (or Ganache), is a direct response to the limitations of existing centralized systems identified in the literature review.

| Feature | Centralized Database | Blockchain (SLR) | Justification for Choice |
| :--- | :--- | :--- | :--- |
| **Integrity** | Susceptible to single-point manipulation and data alteration by privileged administrators. | **Immutable** ledger secured by cryptographic hashing, making historical records tamper-proof. | Directly addresses the problem of **fraud and unauthorized alterations** in land records. |
| **Transparency** | Data access is opaque, restricted to a few officials, fostering distrust and corruption. | **Distributed** ledger allows all authorized stakeholders (e.g., land owners, verifiers) to independently verify the chain of title. | Enhances **public trust and accountability**, a core requirement for developing nations. |
| **Availability** | Vulnerable to system downtime or localized data loss. | **Decentralized** network ensures high availability and resilience against single-point failures. | Mitigates risks associated with **poor infrastructure** and unreliable centralized systems. |

### 1.2 Stakeholders and User Roles

The system is designed to accommodate a multi-tiered access structure to reflect the reality of land administration while leveraging blockchain for core security functions.

| Stakeholder Role | Primary Function | System Access Level |
| :--- | :--- | :--- |
| **Land Owner (Citizen)** | Registering land, initiating transfer, flagging disputes, verifying ownership. | Presentation Layer (Web UI), Read-only access to their own on-chain records. |
| **Land Registrar (Admin)** | Initial land title issuance, approving transfers, resolving disputes, managing user accounts. | Full Application Layer access, Write access to the Smart Contract (via API). |
| **Verifier/Auditor (Legal/Bank)** | Checking the validity and history of a land title for due diligence. | Read-only access to the Application Layer and On-Chain data. |
| **System Administrator** | Backend maintenance, server management, off-chain database integrity. | Application Layer (Backend/DB). |

## 2. Functional Requirements

Functional requirements define the specific actions the system must perform. Each requirement is justified by its necessity in addressing the research gap of fraud, opacity, and poor usability in the target region.

| ID | Requirement | Description | Justification (Addressing Research Gap) |
| :--- | :--- | :--- | :--- |
| **FR1** | **User Registration and Authentication** | Secure registration for all stakeholders with Role-Based Access Control (RBAC). Land Owners require a unique identifier (e.g., National ID hash) and a digital wallet address. | Ensures **accountability** and prevents unauthorized access, a critical security measure against corruption. |
| **FR2** | **Land Title Registration (Initial)** | Registrar initiates the process by recording essential land metadata (e.g., hashed survey coordinates, owner ID hash) and storing the resulting unique Land ID (Token ID) on the blockchain. | Establishes the **immutable genesis record** of ownership, solving the problem of title duplication and manual record loss. |
| **FR3** | **Ownership Transfer** | Owner initiates a transfer request to a new owner's wallet address. The smart contract executes the transfer only after Registrar approval and verification of all required off-chain documents. | Automates the **transfer process transparently** via smart contract logic, reducing bureaucratic delays and potential for fraud. |
| **FR4** | **Land Record Verification** | Any authorized Verifier can query the system using the Land ID to retrieve the current owner's public key and the full transaction history from the blockchain. | Provides **instant, transparent, and tamper-proof verification** of the chain of title, enhancing trust for investors and legal bodies. |
| **FR5** | **Dispute Flagging Mechanism** | Land Owners or Registrars can flag a specific Land ID on the blockchain, temporarily freezing its transferability until an off-chain resolution is logged and approved by the Registrar. | Integrates a mechanism for **community consent and dispute resolution**, a crucial socio-legal requirement for African land tenure systems. |
| **FR6** | **Off-Chain Document Hashing** | The system must generate a cryptographic hash (e.g., SHA-256) of all large off-chain documents (e.g., survey plans, legal deeds) and store only this hash on the blockchain. | Ensures **data integrity** for large files while adhering to the **moderate hardware constraint** by minimizing on-chain data storage. |

## 3. Non-Functional Requirements

Non-functional requirements define the quality attributes of the system, ensuring it is fit for purpose within the specified constraints.

| Requirement | Technical Implementation | Justification |
| :--- | :--- | :--- |
| **Security** | **Cryptographic Hashing** (SHA-256) for all off-chain documents. **Role-Based Access Control (RBAC)** enforced at the Flask API layer and within the Solidity smart contract (`onlyOwner`, `onlyRegistrar` modifiers). | Protects against unauthorized modification and ensures data integrity, addressing the high incidence of fraud. |
| **Privacy** | **Pseudonymity** is maintained by storing only public wallet addresses and cryptographic hashes of personal identifiers on-chain. Sensitive personal data is stored **off-chain** in an encrypted database. | Balances the need for transparency with the necessity of protecting user privacy, aligning with ethical considerations. |
| **Scalability** | Adoption of a **hybrid storage model** (on-chain metadata, off-chain documents) and deployment on a **Permissioned/Consortium** testnet (e.g., Hyperledger Fabric or a private Ethereum chain) is preferred over a public chain. | Optimizes for **prototype-level scalability** and lower transaction costs, mitigating the technical limitation of high gas fees and slow throughput on public chains. |
| **Usability** | **Simple, responsive web interface** (Presentation Layer) designed with a focus on clear workflows and minimal data entry, suitable for users with varying digital literacy. | Directly addresses the **digital literacy gap** and the need for a user-centric design in the target region. |
| **Performance** | **Flask Backend** is lightweight and efficient for handling API requests and database interactions. On-chain operations are limited to essential state changes (ownership, dispute flags) to minimize latency. | Ensures the system remains performant and responsive on **moderate laptop hardware** and low-bandwidth networks. |
| **Reliability** | **Database Replication** (for off-chain data) and the inherent **Byzantine Fault Tolerance** of the blockchain layer ensure continuous operation and data consistency. | Provides resilience against infrastructure failures, a common challenge in developing countries. |

## 4. System Architecture

The SecureLand Registry (SLR) adopts a standard **three-tier architecture** to separate concerns, enhance modularity, and optimize performance, particularly by offloading heavy data and complex business logic from the blockchain.

### 4.1 Architecture Description

**Tier 1: Presentation Layer (Client)**
*   **Components:** Web-based User Interface (UI) accessible via standard web browsers.
*   **Technology:** HTML, CSS, JavaScript (minimalist framework for low-bandwidth optimization).
*   **Function:** Handles user interaction, displays land records, and sends authenticated requests to the Application Layer (Flask API).

**Tier 2: Application Layer (Backend)**
*   **Components:** Python Flask Server, Off-Chain Database (PostgreSQL/SQLite).
*   **Technology:** Python 3.x, Flask framework, Web3.py library, SQLAlchemy (for DB interaction).
*   **Function:** Acts as the central gateway. It handles:
    *   **Authentication and Authorization (RBAC).**
    *   **Business Logic:** Processing user requests, generating cryptographic hashes for documents, and managing the off-chain database.
    *   **Blockchain Interaction:** Using Web3.py to communicate with the Smart Contract Layer (reading data, sending signed transactions).

**Tier 3: Blockchain Layer (Smart Contract)**
*   **Components:** Smart Contract(s) deployed on an Ethereum Testnet (or Ganache).
*   **Technology:** Solidity (for contract logic), EVM (for execution).
*   **Function:** Stores the core, immutable state of land ownership. It is responsible for:
    *   **Land Title Tokenization:** Storing the Land ID, current owner's address, and the hash of the legal document.
    *   **Ownership Transfer Logic:** Enforcing the rules for title transfer.
    *   **Dispute State Management:** Recording and managing dispute flags.

### 4.2 Component Interactions (Data Flow)

1.  **User Request:** A Land Owner requests a title transfer via the Presentation Layer.
2.  **API Call:** The Presentation Layer sends an authenticated request to the Flask API (Application Layer).
3.  **Business Logic:** The Flask API verifies the user's role (FR1), checks the off-chain database for supporting documents, and generates a transaction payload.
4.  **Blockchain Transaction:** The Flask API, acting as a trusted gateway, signs and broadcasts a transaction to the Smart Contract Layer (FR3).
5.  **State Change:** The Smart Contract verifies the transaction, updates the ownership record, and emits an event.
6.  **Data Synchronization:** The Flask API listens for the Smart Contract event and updates the off-chain database with the new transaction hash and status.
7.  **Verification:** A Verifier queries the Flask API (FR4). The API retrieves the current owner's hash from the off-chain DB and cross-references the transaction history directly from the Blockchain Layer, ensuring consistency before displaying the result.

## 5. Smart Contract Design

The core of the system is a single, modular Solidity smart contract, `LandRegistry.sol`, which functions as a tokenized registry for land parcels, similar to an ERC-721 non-fungible token (NFT) but with enhanced access control and dispute mechanisms.

### 5.1 Contract Structure (Pseudocode)

The contract will utilize a `mapping` to store land parcel data and a `struct` to define the immutable properties of a land title.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract LandRegistry {
    // State Variables
    address public registrarAddress; // Address of the trusted Land Registrar
    mapping(uint256 => LandTitle) public landTitles; // Land ID to Title Struct
    uint256 public nextLandId; // Counter for new land titles

    // Struct to define a Land Title
    struct LandTitle {
        address currentOwner;
        bytes32 documentHash; // Hash of the off-chain legal document (FR6)
        uint64 registrationTimestamp;
        bool isDisputed; // Dispute Flag (FR5)
    }

    // Modifiers for Role-Based Access Control (RBAC)
    modifier onlyRegistrar() {
        require(msg.sender == registrarAddress, "Caller is not the Registrar");
        _;
    }

    // 1. Land Registration Logic (FR2)
    function registerLand(address _owner, bytes32 _documentHash) public onlyRegistrar returns (uint256) {
        // Logic: Create a new LandTitle struct and map it to a unique Land ID.
        // This is the genesis transaction, establishing the first owner.
    }

    // 2. Ownership Transfer Rules (FR3)
    function transferOwnership(uint256 _landId, address _newOwner) public {
        // Logic: Requires currentOwner to initiate and Registrar to approve (via a separate function or multi-sig logic).
        // Updates the currentOwner field in the LandTitle struct.
        // Emits a Transfer event.
    }

    // 3. Land Record Verification (FR4)
    function getLandDetails(uint256 _landId) public view returns (address, bytes32, bool) {
        // Logic: Allows anyone to read the current owner, document hash, and dispute status.
    }

    // 4. Dispute Flagging Mechanism (FR5)
    function flagDispute(uint256 _landId) public {
        // Logic: Allows any Land Owner to set the isDisputed flag to true.
        // A separate Registrar function is required to resolve and unflag.
    }
}
```

### 5.2 Emphasis on Core Blockchain Principles

*   **Immutability:** The `documentHash` and `registrationTimestamp` are set during `registerLand` and are never modified, ensuring the integrity of the original record. Only the `currentOwner` and `isDisputed` status are mutable state variables.
*   **Access Control:** The `onlyRegistrar` modifier is critical for preventing unauthorized issuance of new titles (FR2) and ensuring that the system remains a **public permissioned** prototype, suitable for integration with a government body.
*   **Transparency:** The `getLandDetails` function is `public view`, allowing any external party (Verifier) to instantly and trustlessly verify the current owner and the integrity hash of the legal document (FR4).

## 6. Data Management Strategy

The system employs a **hybrid data storage model** to balance the security and immutability of the blockchain with the efficiency and privacy requirements of off-chain storage.

| Data Type | Storage Location | Justification |
| :--- | :--- | :--- |
| **On-Chain (Solidity)** | Ethereum Testnet/Ganache | Stores only **essential, immutable metadata** (Land ID, current owner's public address, cryptographic hash of the legal document, dispute status). |
| **Off-Chain (PostgreSQL/SQLite)** | Flask Backend Database | Stores **sensitive, large, or frequently queried data** (Full personal details, unhashed Land ID, full transaction logs, document URLs, user authentication credentials). |

### 6.1 Rationale for Hybrid Storage

This strategy is chosen to directly address the **moderate hardware constraint** and the **privacy requirement**. Storing large documents (e.g., survey maps) directly on-chain is prohibitively expensive (high gas cost) and inefficient (slow transaction time), which violates the performance and cost non-functional requirements. By storing only the document's hash on-chain (FR6), the system achieves:

*   **Integrity:** The hash acts as a digital fingerprint. Any modification to the off-chain document will result in a mismatch with the on-chain hash, immediately revealing tampering.
*   **Efficiency:** Transaction costs and time are minimized, ensuring the prototype is performant in a low-resource environment.
*   **Privacy:** Sensitive personal data remains in a controlled, encrypted off-chain environment, accessible only via the authenticated Flask API.

### 6.2 Data Integrity and Consistency

Data integrity is maintained through a two-pronged approach:

1.  **On-Chain Integrity:** Guaranteed by the blockchain's cryptographic security and immutability.
2.  **Cross-Layer Consistency:** The Flask Application Layer is responsible for ensuring that every transaction that updates the on-chain state (e.g., `transferOwnership`) is logged in the off-chain database with the corresponding transaction hash and timestamp. When a record is verified (FR4), the API performs a real-time check between the off-chain document's hash and the on-chain hash to confirm consistency.

## 7. Security & Privacy Considerations

Ethical and security considerations are integrated into the design to ensure the system is not only technically sound but also socially responsible, a critical factor for adoption in the Nigerian context.

### 7.1 Role-Based Access Control (RBAC)

Access is strictly controlled at both the Application and Blockchain Layers (FR1).
*   **Application Layer (Flask):** RBAC manages access to the web interface and API endpoints. Only authenticated Registrars can submit transactions that change the on-chain state.
*   **Blockchain Layer (Solidity):** Modifiers like `onlyRegistrar` restrict critical state-changing functions, ensuring that the core registry logic is protected from unauthorized users.

### 7.2 Cryptographic Hashing and Protection

The use of **SHA-256 hashing** for all off-chain documents (FR6) is the primary mechanism for ensuring non-repudiation and integrity. The blockchain protects the integrity of the hash, while the off-chain database protects the confidentiality of the document itself. This separation of concerns is a robust defense against unauthorized modifications.

### 7.3 User Consent Mechanisms and Ethical Alignment

The system design explicitly incorporates mechanisms to address the ethical considerations of privacy and consent (FR4).

*   **Informed Consent:** During user registration (FR1), the Presentation Layer will require explicit, digitally logged consent from the Land Owner before their data (hashed ID, public wallet address) is submitted to the system.
*   **Data Minimization:** Only the minimum necessary data (hashes and public keys) is stored on the public ledger, adhering to the principle of data minimization.
*   **Legal Alignment:** The smart contract logic (e.g., the `transferOwnership` function) is designed to mirror the legal requirements of land transfer in Nigeria (e.g., requiring Registrar approval), ensuring the system is a digital reflection of the local legal reality, not a replacement for it.

## 8. Assumptions and Constraints

This section clearly outlines the foundational assumptions and limitations under which the prototype is designed, which is essential for a defensible academic project.

### 8.1 Legal Assumptions

*   **Legal Recognition of Digital Title:** It is assumed that the hash stored on the blockchain, when combined with the corresponding off-chain document, is legally recognized as sufficient proof of title or a strong evidence of ownership in the target jurisdiction (Nigeria).
*   **Registrar Authority:** It is assumed that the designated Land Registrar's digital signature (private key) is legally recognized as the final authority for the initial issuance of titles and the resolution of disputes.

### 8.2 Technical Limitations and Constraints

*   **Prototype Scope:** The system is a proof-of-concept. It does not account for the full complexity of a national-scale land administration system (e.g., mass data migration, complex cadastral mapping).
*   **Hardware Constraint:** The system is optimized for deployment on a **moderate laptop** (development and testing environment). This necessitates the use of a lightweight backend (Flask) and a hybrid storage model to avoid resource-intensive on-chain operations.
*   **Blockchain Network:** The system will be deployed on a **testnet (e.g., Sepolia, Goerli) or a local development chain (Ganache)**. Performance metrics will be simulated and extrapolated, not measured on a live, high-volume public network.
*   **Off-Chain Security:** The security of the off-chain database (PostgreSQL/SQLite) is assumed to be managed by standard security practices (encryption, firewalls), as the blockchain is only responsible for the integrity of the *hash*, not the confidentiality of the off-chain data.

## 9. Summary

The System Design and Architecture directly addresses the research gap by proposing a **hybrid DApp solution** that is both academically rigorous and practically viable for a low-resource environment. The design moves beyond simple digitization by leveraging the **immutability of Solidity smart contracts** for core ownership logic and employing a **lightweight Python/Flask backend** with a hybrid data strategy to ensure **performance and usability** on moderate hardware. The integration of explicit **RBAC, cryptographic hashing, and consent mechanisms** ensures the system is secure, privacy-preserving, and ethically aligned with the complex socio-legal context of land administration in Nigeria, thereby providing a robust and defensible prototype for the final-year project.

# **MGA Compliant Gaming Platform: Technical Specification & Build Sheet**

**Document Version:** 1.1 **Date:** 2025-09-09 **Status:** For Implementation **Audience:** CTO, Lead Architects, Development & DevOps Teams

## **1.0 Executive Mandate & Core Architectural Principles**

### **1.1 Project Goal**

To design, build, deploy, and maintain a B2C gaming platform and game client that is fully compliant with the regulations set forth by the Malta Gaming Authority (MGA). This document outlines the technical requirements and architectural mandates necessary to achieve and maintain this compliance.

### **1.2 Core Principles (Non-Negotiable)**

All architectural decisions and development efforts must adhere to the following foundational principles:

1. **Compliance-by-Design:** Regulatory requirements are not an afterthought or a QA task. They must be integrated into every stage of the Software Development Lifecycle (SDLC), from initial design to deployment and maintenance.  
2. **Strict Separation of Concerns (Client vs. RGS):** The game client is exclusively a **presentation layer**. It renders UI and captures user input. All determinative game logic, Random Number Generation (RNG), financial transactions, and state management are the sole and exclusive responsibility of the backend Remote Game Server (RGS). Any client-side logic that could influence a game's outcome is a critical compliance failure.  
3. **RGS as the Single Source of Truth:** The RGS is the undisputed authority for all player data, balances, game states, and transaction histories. The client displays data provided by the RGS.  
4. **Immutability of Records:** All regulatory data, especially player activity and financial transactions, must be logged in a complete, accurate, and tamper-evident manner.

## **2.0 System Architecture Overview**

The platform will be a multi-tier, service-oriented architecture designed for security, scalability, and regulatory oversight.

### **2.1 High-Level Components**

* **Game Client:** The player-facing application, built with modern web technologies. Responsible for presentation only.  
* **Operator Backend Platform:** The core of the system, encompassing all server-side logic.  
  * **Web/API Gateway:** Handles initial HTTP/S requests, load balancing, and routing.  
  * **Remote Game Server (RGS):** The stateful service containing all game logic, RNG, and session management. Communicates with the client via WebSocket Secure (WSS).  
  * **Wallet & Payments Service:** Manages the player's "Seamless Wallet" and integrates with third-party Payment Service Providers (PSPs).  
  * **Player Account Service:** Manages user authentication, profiles, and responsible gaming settings.  
  * **Logging & Auditing Service:** Centralized service for ingesting and securely storing all regulatory log data.  
* **Primary Database Cluster:** Stores all operational data (player accounts, transactions, etc.).  
* **MGA Replication Server:** A physically distinct server located in Malta for real-time replication of all regulatory data.  
* **Third-Party Services:**  
  * Payment Service Provider (PSP) \- PCI DSS Level 1 Compliant.  
  * MGA-Approved Auditors & Testing Labs.

### **2.2 Data & Communication Flow**

1. **Authentication:** Client communicates with the Player Account Service via HTTPS to log in and receive a JWT.  
2. **Gameplay:** Client establishes a persistent WSS connection to the RGS, authenticated with the JWT. All game actions are sent over this channel.  
3. **Transactions:** The RGS orchestrates all financial operations (bets, wins) against the Wallet Service, which updates the Primary Database.  
4. **Data Replication:** All committed transactions in the Primary Database are replicated in real-time to the MGA Replication Server in Malta.

## **3.0 Foundational Infrastructure Specification**

### **3.1 Primary Hosting Environment**

* **Geographic Location:** The primary technical infrastructure must be physically located within **Malta or a European Economic Area (EEA) member state**.  
* **Data Center Certification:** The chosen hosting provider's facility **must be ISO/IEC 27001:2013 certified**. Proof of certification is required for the MGA application.  
  * *Vendor Shortlist:* Digital Realty, Equinox, Vantage, or other providers with certified EEA facilities.  
* **Cloud Environment Policy:**  
  * Public, private, and hybrid cloud models are permissible.  
  * **Critical Components** (RNG, Jackpot servers, Player & Financial databases) are under high scrutiny. These components **must be hosted in a private or logically isolated environment** (e.g., a Virtual Private Cloud) with a comprehensive risk assessment provided. A shared/multi-tenant public cloud environment is not acceptable for these core systems.

### **3.2 MGA Real-Time Data Replication Server**

* **Requirement:** This is a **critical, non-negotiable mandate** if the primary infrastructure is hosted outside of Malta.  
* **Physical Location:** A physical server must be located **within Malta**.  
* **Connectivity:** A high-capacity, secure, and low-latency network link (e.g., VPN over a dedicated line) must be established between the primary data center and the Malta replication server.  
* **Replication Technology:** The chosen database technology must support synchronous or near-synchronous replication to ensure data is mirrored in real-time. The solution must be robust and monitored as a production-critical component.  
* **MGA Access:** Documented procedures must be in place to grant MGA officials immediate and unrestricted physical and electronic access to this server.  
* **Required Documentation:** A detailed proposal including server location (rack number), IP address, network schematics, and replication methodology must be submitted to the MGA.

## **4.0 Frontend Game Client Specification**

### **4.1 Technology Stack**

* **Core Framework:** **React or Vue.js** (Component-based architecture is essential).  
* **Language:** **TypeScript** (Strongly recommended for type safety and maintainability).  
* **Game Rendering Engine:**  
  * For 2D Games: **Phaser.js** or **Pixi.js**.  
  * For 3D Games: **Babylon.js** or **Three.js**.  
* **Communication:**  
  * **HTTPS:** **Axios** library for RESTful API calls.  
  * **WSS:** Native browser WebSocket API or a reliable library (e.g., Socket.IO configured for WSS-only).  
* **Styling:** CSS3 with a responsive framework (**Tailwind CSS** or similar).  
* **Build Tools:** **Vite** or **Webpack**.

### **4.2 Client Architecture**

The client application must be structured into distinct layers:

1. **Presentation Layer:** Renders all UI, game graphics, animations, and sound. Implements all MGA-mandated display elements.  
2. **Communication Layer:** Manages all secure communication (HTTPS/WSS) with the backend. Handles API requests, responses, and error handling.  
3. **State Management Layer:** Manages the client's local state (e.g., UI state). Utilizes a formal state management library (e.g., **Redux, Vuex, Zustand**) to handle data received from the RGS, ensuring UI elements like the player balance are always in sync with the authoritative server state.

### **4.3 Mandatory UI & Display Requirements**

| UI Element | Requirement | Implementation Priority |
| :---- | :---- | :---- |
| **Real-Time Balance** | Must be permanently visible on all game screens. Must update instantly after every bet/win via a push message from the RGS. Must include the correct currency symbol. | **CRITICAL** |
| **Real-Time Clock** | A clock showing the player's local time must be permanently visible, especially in full-screen mode. | **CRITICAL** |
| **Game Exit Button** | A clear, prominent, and easily accessible control to exit the game and return to the lobby must always be present. | **CRITICAL** |
| **Game Rules Access** | A link or button to access the complete rules of the game being played must be no more than one click away from the main game interface. | **CRITICAL** |

### **4.4 Player Protection Features (UI Implementation)**

The client must provide a clear, user-friendly interface for all Player Protection Directive tools. This section must be easily accessible from the main account interface.

* **Financial Limits:**  
  * Interface to set **Deposit** and **Wagering** limits (daily/weekly/monthly).  
  * UI must clearly explain that tightening a limit is **immediate**.  
  * UI must clearly explain that loosening or removing a limit is subject to a **24-hour cooling-off period**. The client should reflect the pending change.  
* **Reality Check:**  
  * Interface for the player to set a time interval (e.g., 30, 60, 90 mins).  
  * When triggered, the client **must suspend all gameplay** with a modal overlay.  
  * The modal must display: session duration, total wagers, and net win/loss for the session.  
  * The modal must have two explicit options: "Continue" or "Exit Game". Gameplay cannot resume until one is clicked.  
* **Self-Exclusion:**  
  * A clear and unambiguous interface for players to self-exclude.  
  * Options must include definite periods (6 months, 1 year, etc.) and an indefinite period.  
  * The process must be **immediate**. Upon final confirmation, the client must terminate the player's session and log them out.

## **5.0 Backend (RGS & Platform) Specification**

### **5.1 API Architecture**

* **Stateless Operations (Login, Account Mgmt):** **RESTful API over HTTPS**.  
* **Stateful Operations (Gameplay):** **WebSocket Secure (WSS)** for persistent, low-latency, bidirectional communication.  
* **Data Format:** **JSON** for all requests and responses.  
* **API Security:**  
  * **Authentication:** All authenticated endpoints must be protected. The server must issue a **JSON Web Token (JWT)** upon login, which must be validated (signature, expiration) on every subsequent request.  
  * **Transport Security:** TLS 1.2 or higher must be enforced for all connections.  
  * **Origin Header Validation:** The WSS handshake must perform strict Origin header checking to prevent CSWH.  
  * **Input Sanitization:** All data received from the client must be rigorously validated and sanitized on the server before processing.

### **5.2 "Seamless Wallet" Transaction Flow (CRITICAL)**

This model is mandatory. Every bet is an atomic transaction against the central player wallet.

1. **Client \-\> RGS (WSS):** placeBetRequest { transactionId: "uuid-v4-client", gameId: "game-abc", betAmount: 1.00 }  
2. **RGS:**  
   * Validates JWT and player session.  
   * Checks transactionId for idempotency. If seen before, return "already processed" error.  
   * Calls Wallet Service to debit betAmount.  
   * If debit is successful, calls certified RNG to get the game outcome.  
   * Calculates win amount based on outcome.  
   * If win \> 0, calls Wallet Service to credit winAmount.  
   * Logs the complete transaction (bet and result) with the same transactionId.  
3. **RGS \-\> Client (WSS):** gameResultResponse { transactionId: "uuid-v4-client", outcome: \[...\], winAmount: 50.00, newBalance: 159.50 }

### **5.3 Session Management**

* The server must enforce a hard **30-minute inactivity timeout** on all player sessions.  
* The session token (JWT) must be invalidated on the server-side after this period. Any API call with an expired token must be rejected.

### **5.4 Regulatory Data Logging Matrix (CRITICAL)**

The RGS and backend platform are responsible for logging every significant event to a secure, tamper-evident store. The following matrix defines the minimum data points required.

| Event Name | Trigger | Minimum Data to Log |
| :---- | :---- | :---- |
| **Player Login/Logout** | Authentication attempt or session end | PlayerID, SessionID, Timestamp (UTC), IP Address, User-Agent, Status (Success/Fail), Logout Reason |
| **Game Round Start (Bet)** | Player initiates a wager | PlayerID, SessionID, Timestamp (UTC), GameID, **UniqueTransactionID**, BetAmount, Pre-TransactionBalance, Currency |
| **Game Round End (Result)** | RGS determines outcome | PlayerID, SessionID, Timestamp (UTC), GameID, **UniqueTransactionID**, RNG Result (raw), WinAmount, Post-TransactionBalance, Currency |
| **Financial Deposit** | Player adds funds | PlayerID, Timestamp (UTC), **UniqueTransactionID**, Amount, PaymentMethod, Pre-Balance, Post-Balance, Status |
| **Financial Withdrawal** | Player requests funds | PlayerID, Timestamp (UTC), **UniqueTransactionID**, Amount, DestinationAccount, Pre-Balance, Post-Balance, Status (Requested/Approved) |
| **RG Limit Change** | Player modifies a limit | PlayerID, Timestamp (UTC), IP Address, LimitType, OldValue, NewValue |
| **Self-Exclusion** | Player requests exclusion | PlayerID, Timestamp (UTC), IP Address, ExclusionType, Duration |

## **6.0 Security & Compliance Specification**

### **6.1 Secure SDLC (ISO 27001 Annex A.14)**

The following practices must be integrated into the development workflow:

* \[ \] **Secure Development Policy:** A formal, documented policy governing the SDLC must be created and enforced.  
* \[ \] **Code Reviews:** Mandatory, peer-reviewed pull requests for all code changes, with a security checklist.  
* \[ \] **Static Analysis (SAST):** Integrate automated security analysis tools into the CI/CD pipeline.  
* \[ \] **Dependency Scanning:** Continuously scan all third-party libraries and frameworks for known vulnerabilities.  
* \[ \] **Environment Segregation:** Strict separation of Development, Testing/Staging, and Production environments. Production data (even anonymized) is not permitted in lower environments.  
* \[ \] **Change Control:** Formal, documented procedures for all changes to the production environment.

### **6.2 PCI DSS Level 1 Compliance**

* **Strategy:** The platform will **reduce PCI DSS scope** by outsourcing all payment card processing to a certified Level 1 Payment Service Provider (PSP).  
* **Implementation:** The client will integrate the PSP's payment form using a **secure iframe or a redirect**. At no point shall any cardholder data (PAN, CVV) be transmitted to or stored on operator-controlled systems.  
* **Residual Compliance:** Even with outsourcing, the platform must still adhere to PCI DSS requirements related to network security, secure configurations, access control, and logging. Regular vulnerability scanning (quarterly) and penetration testing (annually) are still mandatory.

## **7.0 Testing, Verification & Go-Live Plan**

This project follows a gated, hybrid development model. Proceeding to the next phase is contingent on successfully clearing the gates of the current phase.

### **Phase 1: Development & Internal QA**

* Standard agile development sprints.  
* Comprehensive unit, integration, and end-to-end testing.  
* Internal security reviews and code audits.

### **Phase 2: Mandatory Third-Party Auditing (Pre-Launch)**

* **Engage MGA-Approved Auditors EARLY.**  
* **Penetration Test:** A full-scope penetration test of the entire platform (client, APIs, infrastructure) must be conducted and passed.  
* **Game & RNG Certification:** An MGA-approved testing lab (e.g., GLI, eCOGRA) must certify the game logic, fairness, and RNG implementation.  
* **Platform Compatibility Test:** The client must be formally tested and verified across all supported browsers and devices.

### **Phase 3: MGA Official Audits (Go/No-Go Gates)**

* **System Audit (Pre-Launch):** After deployment to the production environment, an MGA-approved auditor will conduct a full system audit. This audit verifies that the live system **perfectly matches** all submitted architectural and policy documentation.  
* **Compliance Audit (Post-Launch):** Conducted after the first year of operation. This reviews ongoing adherence to all MGA regulations, focusing on player protection, logging, and financial reporting in practice.

## **8.0 Documentation Requirements**

All technical documentation must be treated as **living documents**, version-controlled alongside the source code, and updated with every significant system change.

### **Required Documents for MGA Submission:**

1. **System Architecture Documentation:**  
   * Network Topology Diagrams  
   * Hardware/VM Specifications  
   * Geographic Locations & IP Address Ranges  
2. **Application Architecture Documentation:**  
   * Application Inventory & Versions  
   * Server Hosting Map for each Application  
   * Detailed Interaction & Data Flow Diagrams (especially Client \<-\> RGS)  
3. **Security Policies & Procedures:**  
   * Information Security Policy (based on ISO 27001\)  
   * Change Management Procedures  
   * Incident Response Plan  
4. **Data Replication Plan** (if applicable)

## **9.0 Implementation, Governance & Next Steps**

This section outlines the operational approach to ensure the technical specifications are implemented correctly and that a culture of compliance is maintained.

### **9.1 Critical Compliance Task Checklist**

This checklist summarizes the highest-priority, non-negotiable tasks that form the critical path to a compliant launch.  
**Phase: Planning & Procurement (Immediate Priority)**

* \[ \] Select and contract with an ISO 27001:2013 certified data center in the EEA.  
* \[ \] If hosting outside Malta, select and contract a provider for the physical replication server in Malta.  
* \[ \] Select and engage with MGA-approved third-party partners for security testing (pen-testing) and game/RNG certification.

**Phase: Architecture & Design (Pre-Development)**

* \[ \] Finalize and formally document the complete system and application architecture diagrams for MGA submission.  
* \[ \] Design and document the secure network topology, including firewall rules and isolation for critical components.  
* \[ \] Design and document the real-time data replication solution, including failure and recovery modes.  
* \[ \] Design all financial API endpoints to be idempotent using unique transaction IDs.

**Phase: Development & Implementation**

* \[ \] Enforce HTTPS/WSS for all client-server communication from the first line of code.  
* \[ \] Implement the secure token-based (JWT/OAuth2) authentication and session management system with the mandatory 30-minute player inactivity timeout.  
* \[ \] Architect the client-server interaction strictly following the "Seamless Wallet" model.  
* \[ \] Implement the full suite of Player Protection Directive tools: immediate self-exclusion, enforceable financial limits (with 24-hour cool-off), and gameplay-suspending reality checks.  
* \[ \] Implement comprehensive server-side logging for every event detailed in the Regulatory Data Logging Matrix.

### **9.2 Establishing a Compliant Development Workflow**

Process is as important as technology for MGA compliance. The development workflow must be structured, documented, and secure.

* **Code Quality and Reviews:** Institute a mandatory code review process for all changes. This process must include a specific checkpoint where the reviewer verifies that the code adheres to security best practices (e.g., input validation, proper error handling) and does not violate any compliance rules.  
* **Version Control:** Employ a disciplined version control strategy, such as GitFlow. This ensures a clear separation between feature development (feature branches), integration (develop branch), pre-release stabilization (release branches), and production code (main or master branch). This structure prevents untested or unapproved code from accidentally being deployed.  
* **Configuration Management:** Enforce a strict policy against hard-coding any secrets (API keys, passwords, certificates) or environment-specific configurations (IP addresses, domain names) in the source code. Utilize a secure configuration management system or environment variables to inject these values at build or run time. This is critical for maintaining the required separation and security of development, staging, and production environments.  
* **Documentation Maintenance:** All technical documentation, particularly the System Architecture diagrams and API specifications, must be treated as living documents. They must be version-controlled alongside the code and updated as part of the release process for any significant change. This ensures that the documentation provided to the MGA always reflects the current state of the live system.

### **9.3 Next Steps**

1. **Review & Approval:** This document is to be reviewed and formally approved by the CTO and lead architects.  
2. **Team Briefing:** Conduct a full briefing with the development, DevOps, and QA teams to ensure universal understanding of the compliance-by-design methodology.  
3. **Vendor Engagement:** Immediately begin the procurement and engagement process for data centers and third-party auditors as outlined in the checklist.  
4. **Project Plan:** Translate the phases and tasks outlined in this specification into a detailed project plan with timelines, resource allocation, and milestones tied to the mandatory audit gates.
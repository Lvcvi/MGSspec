# **MGA Compliant Game Client: Final Architectural Specification**

**Document Version:** 2.0 (Final) **Date:** 2025-09-09 **Status:** **LOCKED \- FOR IMPLEMENTATION** **Audience:** Frontend Development Team, Lead Frontend Architect, QA Team

## **Document Control**

## **This document provides the complete architectural and implementation specification for the frontend game client. It is derived from and must remain consistent with the master MGA\_Compliance\_Build\_Sheet\_FINAL.md. All frontend development must strictly adhere to the patterns and requirements defined herein.**

## **1.0 Core Architectural Mandate & Guiding Principles**

### **1.1 The Prime Directive**

The game client is a **secure presentation layer only**. It renders the UI and game state as dictated by the authoritative Remote Game Server (RGS). It must not contain any determinative game logic.

### **1.2 Guiding Principles for Development**

1. **Server-Authoritative State:** The client's state is a temporary mirror of the RGS.  
2. **Fail Securely:** On connection loss, the client must enter a locked state, preventing gameplay until re-synchronized with the RGS.  
3. **Assume Zero Trust:** Validate user input for UX, but rely solely on server-side validation.  
4. **Idempotency is Mandatory:** Every financial action (bet) must be sent with a unique transactionId.

## **2.0 High-Level Client Architecture**

The client is a Single-Page Application (SPA) built with a modern component-based framework (React/Vue.js). The architecture is modular to ensure a clear separation of concerns.

## **3.0 Module Specification & Implementation Details**

### **3.1 CoreApp Module**

**Responsibility:** The main application shell and orchestrator.  
**Key Components & Logic:**

* App.tsx: Root component. Initializes services on mount.  
* MainLayout.tsx: Contains persistent UI like the header, footer, and ComplianceBar.  
* **Session Inactivity Timer:**  
  * On login, a 30-minute inactivity timer must be initiated.  
  * Any user interaction (click, keypress, touch) must reset this timer.  
  * At 28 minutes, a warning modal must be displayed.  
  * On expiry, the client must call ApiService.logout(), clear all local state and storage, and redirect to the login page.

### **3.2 ApiService Module**

**Responsibility:** The single, secure gateway for all network communication.  
**Implementation Details:**

* **HTTPS Service (https://.ts):**  
  * **Request Interceptor:** Must automatically attach the Authorization: Bearer \<JWT\> header.  
  * **Response Interceptor:** Must handle global HTTP error codes:  
    * 401 Unauthorized: Intercept, dispatch a global logout action, and redirect to login.  
    * 5xx Server Error: Display a generic error and log details.  
* **WebSocket Service (websocket.ts):**  
  * connect(token): Must initiate a **WSS** connection.  
  * send(payload):  
    * **Idempotency Key:** Before sending a bet, this method **must** generate a UUID v4 (transactionId) and include it in the payload.  
* **Connection Management:**  
  * Must implement an exponential backoff retry mechanism for dropped connections.  
  * During a disconnected state, a global flag isConnectionLost must be set in the state store, causing the UI to lock.  
  * Upon reconnection, the client must request a full state synchronization from the RGS.

### **3.3 StateService Module**

**Responsibility:** The centralized state management store (e.g., Redux, Vuex).  
**Detailed State Slices:**

* session: { isAuthenticated, token, user, sessionStartTime, isSessionExpired }  
* player: { balance: number, currency: string }  
* game: { gameId, isGameSuspended: boolean, isConnectionLost: boolean, currentRound }  
* responsibleGaming: { limits, limitChangeCooldown, realityCheck, sessionStats: { totalWagered, netWinLoss } }

### **3.4 ComplianceUI Module**

**Responsibility:** A suite of "dumb" components that render UI based on data from the StateService.  
**Key Components & Required Logic:**

* ComplianceBar.tsx:  
  * BalanceDisplay: Renders player.balance formatted with player.currency. Must update in real-time.  
  * Clock: Displays the user's system time, ticking every second.  
* RealityCheckModal.tsx:  
  * **Trigger Logic:** A timer in CoreApp checks every second if (now \- realityCheck.lastCheckTimestamp) \> realityCheck.interval. If true, it dispatches an action to set game.isGameSuspended \= true.  
  * **Rendering:** When isGameSuspended is true, this modal must render, overlaying the entire application.  
  * **Data Display:** It must display sessionStats.totalWagered and sessionStats.netWinLoss.  
  * **Actions:** "Continue" button calls ApiService.acknowledgeRealityCheck(). "Exit Game" button navigates to the lobby.  
* LimitsManager.tsx:  
  * **Logic:**  
    * **Tightening Limit:** Sends new value. UI updates on success.  
    * **Loosening Limit:** Sends new value. On success, the RGS will reply with a 24-hour expiry timestamp. The client stores this in limitChangeCooldown and displays a countdown message.  
* SelfExclusion.tsx:  
  * **Logic:** Upon final confirmation, the client calls ApiService.initiateSelfExclusion(). The server will immediately terminate the session, and the ApiService's global 401 handler will log the user out.

### **3.5 GameEngine Module**

**Responsibility:** Manages game-specific rendering, assets, and input.  
**Compliance-Specific Requirements:**

* **Gameplay Suspension:** The main game loop **must** check game.isGameSuspended and game.isConnectionLost on every frame. If either is true, the loop must be paused, and all user input within the canvas must be ignored.  
* **No State Logic:** The engine receives the game outcome from the StateService and is only responsible for visualizing it.

### **3.6 PaymentIntegration Module**

**Responsibility:** To handle deposits and withdrawals in a PCI DSS compliant manner.  
**Implementation:**

* This module **must not** contain any form fields for entering credit card details.  
* It will render a secure **iframe** provided by the third-party PSP.  
* It will listen for postMessage events from the iframe to know when the transaction is complete.

## **4.0 Developer Compliance Checklist (For Code Reviews)**

* \[ \] **Authentication:** Is the JWT used? Is 401 globally handled?  
* \[ \] **Session:** Is the 30-min inactivity timer fully implemented?  
* \[ \] **Transactions:** Does every bet generate a unique transactionId?  
* \[ \] **State:** Is all critical UI data read from the central StateService?  
* \[ \] **Compliance UI:**  
  * Is the Reality Check modal fully suspensive?  
  * Does the Limits UI correctly handle the 24-hour cool-down?  
  * Is the self-exclusion flow immediate and final?  
* \[ \] **Security:** No hardcoded secrets? Is communication strictly HTTPS/WSS?  
* \[ \] **Error Handling:** Does the UI correctly lock on WebSocket connection loss?  
* \[ \] **Logging:** Does every significant user action trigger a corresponding ApiService call?
# near-agent-escrow-pack

**DAiL Hackathon Entry Pack — NEAR Agent Economy Track (bnty_0451)**  
*Based on the original DAiL build `near-agent-escrow-market` (bnty_0397 by Fuma2514).*

A judge-ready submission pack demonstrating an escrowed agent-services market modeled on NEAR Intents: buyer funds are locked into an escrow contract at intent time, work is verified upon delivery, and escrow releases to the seller on acceptance or refunds in full upon dispute.

---

## 1. Quickstart: Reproduce in Under 60 Seconds ("Do X, See Y")

This project requires **only Node.js (v18+)** with zero package installations (`npm install` is NOT required).

### Step 1: Run the Interactive Lifecycle Demo
**Do X:**
```bash
node demo.js
```

**See Y:**
```text
============================================================
▶ STEP 1: INITIALIZE MARKET & REGISTER AGENTS
============================================================
Registered agents:
 - alice-agent   : 100 NEAR
 - bob-agent     : 50 NEAR
 - charlie-agent : 100 NEAR

============================================================
▶ STEP 2: LIST SERVICES
============================================================
Listed services by bob-agent: 'svc_clean' (30 NEAR), 'svc_audit' (40 NEAR)

============================================================
▶ STEP 3: HAPPY PATH (Alice hires Bob)
============================================================
Action: Alice hires Bob for 'svc_clean' (30 NEAR)
Order created: ord_happy_1
Balance after hire: Alice = 70 NEAR, Bob = 50 NEAR
Invariant check: locked = 30 NEAR, openOrders = 1

Action: Bob delivers work
Order state: delivered

Action: Alice accepts delivery
Balance after accept: Alice = 70 NEAR, Bob = 80 NEAR
Invariant check: locked = 0 NEAR, openOrders = 0

============================================================
▶ STEP 4: DISPUTE & REFUND PATH (Charlie hires Bob)
============================================================
Action: Charlie hires Bob for 'svc_audit' (40 NEAR)
Order created: ord_dispute_1
Balance after hire: Charlie = 60 NEAR (40 locked in escrow)
Invariant check: locked = 40 NEAR, openOrders = 1

Action: Bob delivers low-quality work

Action: Charlie disputes delivery
Balance after dispute refund: Charlie = 100 NEAR (100% refunded)
Bob balance remains unaffected: Bob = 80 NEAR
Invariant check: locked = 0 NEAR, openOrders = 0

============================================================
▶ STEP 5: AUDIT LOG (EVENT TRAIL)
============================================================
Total lifecycle events recorded: 11
...
============================================================
▶ DEMO SUMMARY: ALL INVARIANTS SATISFIED (OK)
============================================================
✓ Zero fund leakage: locked tokens strictly returned to 0 upon completion
✓ Two-exit resolution proven: Release -> seller (+30), Dispute -> buyer (+40)
✓ Ready for judge evaluation.
```

---

### Step 2: Run the Automated Test Suite
**Do X:**
```bash
node test_market.js
```

**See Y:**
```text
============================================================
RUNNING NEAR AGENT ESCROW TEST SUITE
============================================================
  ✓ PASS: 1. Happy path: Hire -> Deliver -> Accept releases funds to seller
  ✓ PASS: 2. Dispute path: Dispute after delivery returns 100% of escrow to buyer
  ✓ PASS: 3. Insufficient balance rejects hire without modifying ledger
  ✓ PASS: 4. Invalid state transition: Cannot accept an undelivered order
  ✓ PASS: 5. Invalid state transition: Cannot double-deliver an order
  ✓ PASS: 6. Unknown service or unknown buyer throws descriptive error
  ✓ PASS: 7. Invariant multi-order concurrency check
------------------------------------------------------------
RESULT: 7/7 tests passed.
ALL TESTS COMPLETED SUCCESSFULLY (EXIT 0).
```

---

## 2. Full List of Libraries, Templates, and Boilerplate Used

- **Runtime Environment:** Node.js v18+ (tested on v20/v22).
- **Libraries Used:** 
  - `node:crypto` (standard library only) — used for order ID generation and hashing.
  - `node:assert` (standard library only) — used for test assertions in `test_market.js`.
- **Third-Party NPM Dependencies:** **0 (None)**. No `npm install`, zero supply-chain risk, zero external network downloads required during test execution.
- **Templates & Boilerplate:**
  - Upstream prototype logic adapted from [`Fuma2514/near-agent-escrow-market`](https://github.com/Fuma2514/near-agent-escrow-market) (DAiL bnty_0397).
  - Enhanced with explicit parameter validation, deterministic error types, dispute reasons, and strict invariant checking.

---

## 3. Disclosure: Simulated vs Deployed Functionality

To ensure full transparency for hackathon reviewers and judges:

| Capability | Status | Implementation Details |
|---|---|---|
| **Multi-Agent State Machine** | **Fully Executable & Tested** | In-memory ledger tracking agent balances, active listings, order statuses (`escrowed` → `delivered` → `released` / `refunded`). |
| **Invariant Verification** | **Fully Executable & Tested** | `verify()` validates that total circulating tokens + locked escrow strictly equals initial system issuance. |
| **Two-Exit Dispute Resolution** | **Fully Executable & Tested** | Full refund logic on dispute without seller penalty or fund leakage. |
| **Audit Event Logging** | **Fully Executable & Tested** | Monotonically timestamped JSON event log for all state transitions. |
| **NEAR NEP-141 Token Transfers** | *Simulated Interface* | Modeled after `ft_transfer_call` semantics where tokens transfer to the contract upon intent submission. |
| **NEAR Intents Solver Settlement** | *Simulated Interface* | Modeled off-chain: buyer publishes intent to hire; upon acceptance, contract executes standard payout. |
| **RPC & Explorer Links** | *Simulated / Local Sandbox* | The reference implementation runs deterministically offline; no live testnet gas or faucet tokens required for verification. |

---

## 4. Architecture & State Transition Model

```
               [ hire(buyer, svc) ]
 [REGISTERED] ---------------------> [ ESCROWED ]
                                          |
                                          | [ deliver(orderId) ]
                                          v
                                     [ DELIVERED ]
                                     /           \
               [ accept(orderId) ]  /             \  [ dispute(orderId) ]
                                   v               v
                             [ RELEASED ]     [ REFUNDED ]
                             (Paid to seller)  (Refunded to buyer)
```

1. **`hire(buyer, serviceId)`**: Debits `svc.price` from buyer balance; credits escrow bucket.
2. **`deliver(orderId)`**: Seller flags task as complete; escrow remains locked.
3. **`accept(orderId)`**: Buyer confirms delivery quality; funds transfer from escrow to seller.
4. **`dispute(orderId)`**: Buyer rejects low-quality deliverable; funds transfer from escrow back to buyer.

---

## 5. Judge Verification Summary

- **Repository:** https://github.com/DSP-AID/near-agent-escrow-pack
- **Deliverable Files:**
  - `market.js` — Core protocol engine
  - `demo.js` — Step-by-step reproducible scenario demo
  - `test_market.js` — 7-case automated test suite
  - `README.md` — Complete judge-ready specification
- **Verification Command:** `node demo.js && node test_market.js` (exits 0 with all checks green).

# Feature Spec: Web3 Payment Gateway

**Product:** Lab3 Platform  
**Feature:** Multi-channel Payment — Stripe + Web3 Wallet  
**Version:** 2.0  
**Author:** QA Team  
**Date:** 2026-02-24

---

## 1. Overview

Users can complete purchases on the Lab3 platform via two payment channels:
- **Fiat:** Stripe (credit/debit card)
- **Web3:** Crypto wallets — MetaMask, Phantom, Rabby

A payment is considered successful only when the backend confirms the transaction hash (Web3) or payment intent (Stripe) with status `confirmed`.

---

## 2. Supported Payment Methods

| Method | Type | Network | Currency |
|--------|------|---------|----------|
| Stripe | Fiat | N/A | USD, VND |
| MetaMask | Web3 Wallet | Ethereum, Polygon | ETH, MATIC, USDC |
| Phantom | Web3 Wallet | Solana | SOL, USDC |
| Rabby | Web3 Wallet | Ethereum, BSC, Arbitrum | ETH, BNB, ARB |

---

## 3. Order State Machine

### 3.1 Order States

| State | Description |
|-------|-------------|
| `draft` | Order created, payment not yet initiated |
| `processing` | Payment initiated, awaiting on-chain inclusion |
| `processing_finalizing` | Tx included in 1 block, awaiting required block confirmations |
| `confirmed` | Required block confirmations reached (or Stripe succeeded) |
| `failed` | Payment failed (declined, rejected, or error) |
| `timeout` | No Phase 1 detection within 45s polling window |
| `refund_pending` | Refund initiated, awaiting processing |
| `refunded` | Refund completed successfully |
| `partially_refunded` | Partial refund completed |
| `chargebacked` | Chargeback filed by user via bank |
| `cancelled` | Order cancelled before payment confirmed |
| `frozen` | Account frozen due to re-org double-spend risk — manual review required |

### 3.2 State Transitions

```
draft → processing              (user initiates payment)
processing → processing_finalizing  (tx included in 1 block)
processing_finalizing → confirmed   (required block confirmations reached)
processing → failed             (payment declined / rejected)
processing → timeout            (45s polling, no Phase 1 detection)
timeout → processing_finalizing (late Phase 1 detected in background monitoring)
timeout → failed                (manual review: tx dropped)
processing_finalizing → frozen  (re-org detected + product already delivered)
processing_finalizing → processing  (re-org detected + product not yet delivered)
confirmed → refund_pending      (refund initiated)
refund_pending → refunded       (refund completed)
refund_pending → partially_refunded  (partial refund completed)
confirmed → chargebacked        (bank chargeback received)
draft → cancelled               (user cancels before paying)
processing → cancelled          (user cancels, no tx hash yet)
```

### 3.3 Retry Logic

- **BR-016:** If order is in `failed` state → user may retry payment (creates new payment attempt, same order ID)
- **BR-017:** If order is in `timeout` state → system checks tx hash one final time before allowing retry. If tx found confirmed → update to `confirmed`, do NOT allow retry.
- **BR-018:** Maximum 3 retry attempts per order. On 4th failure → order permanently `failed`, require user to create new order.
- **BR-019:** Retry interval must be at least 10 seconds between attempts to prevent spam.

---

## 4. User Flow

### 4.1 Stripe Flow
1. User selects item → clicks "Pay with Card"
2. System creates Stripe Payment Intent with idempotency key = `order_id + attempt_number`
3. User enters card details (number, expiry, CVV)
4. Stripe processes → returns `payment_intent.status`
5. If `succeeded` → order `confirmed`, redirect to success page
6. If `payment_failed` → order `failed`, show Stripe error message
7. If `requires_action` → trigger 3D Secure flow → if auth fails → order `failed`

### 4.2 Web3 Wallet Flow
1. User selects item → clicks "Pay with Wallet"
2. System detects installed wallets, shows available options
3. User selects wallet (MetaMask / Phantom / Rabby)
4. System requests wallet connection → wallet popup appears
5. User approves connection
6. System calculates crypto amount using real-time exchange rate (see Section 7)
7. System estimates gas fee → displays to user before confirmation
8. System sends transaction request: recipient address, amount, network
9. User reviews and confirms transaction in wallet
10. System stores tx hash → begins polling every 3 seconds, max 15 retries (45 seconds)
11. If confirmed on-chain with required block confirmations → order `confirmed`
12. If timeout → order `timeout` → continue background monitoring for 10 minutes (see BR-022)

---

## 5. Business Rules

### 5.1 Payment Amount
- **BR-001:** Minimum payment amount is $1.00 USD or equivalent in crypto
- **BR-002:** Maximum payment amount per transaction is $10,000 USD or equivalent in crypto
- **BR-003:** If wallet balance is insufficient → block transaction, show "Insufficient balance. Your wallet has [X token], required [Y token]."

### 5.2 Wallet & Connection
- **BR-004:** If user rejects wallet connection → show "Connection rejected. Please try again."
- **BR-005:** If user rejects transaction in wallet → order `failed`, show "Transaction rejected by user."
- **BR-006:** If wallet disconnects mid-flow → show "Wallet disconnected. Please reconnect and try again." Order remains `processing` for 60 seconds before moving to `failed`.
- **BR-007:** If user closes browser during `processing` → order remains `processing`. On next visit, system checks tx hash status and updates order accordingly.
- **BR-008:** If user opens 2 tabs with same order → second tab must be blocked with "Payment already in progress for this order."

### 5.3 Transaction Timing & Confirmation
- **BR-009:** Transaction polling: every 3 seconds, max 15 retries = 45 seconds total
- **BR-010:** Two-phase confirmation model:
  - **Phase 1 — Included (1 block):** Tx appears on-chain → order moves to `processing_finalizing`, UI shows "Confirmed (finalizing...)"
  - **Phase 2 — Finalized (sufficient blocks):** Required blocks reached → order moves to `confirmed`
  - Required blocks per network:
    - Ethereum: 12 blocks (~2.5 minutes)
    - Polygon: 128 blocks (~4 minutes)
    - Solana: 31 slots (~13 seconds)
    - BSC: 15 blocks (~45 seconds)
    - Arbitrum: 1 block (~0.3 seconds)
  - **Note:** 45s polling window detects Phase 1 (tx included). Phase 2 finalization always runs in background regardless of polling timeout.
- **BR-011:** If no Phase 1 detection within 45s → order moves to `timeout`. Background monitoring continues every 30s for 10 minutes for both phases.
- **BR-022:** If Phase 1 or Phase 2 confirmed after 45s window → order updated from `timeout` to `processing_finalizing` or `confirmed` respectively. User notified via email/notification.

### 5.4 Blockchain Edge Cases
- **BR-023:** **Chain re-org:** If a confirmed tx is invalidated by chain re-org:
  - If product/digital asset already delivered → **freeze account** immediately, trigger manual review. Do NOT revert order to `processing` to prevent double-spend.
  - If product not yet delivered → revert order to `processing`, re-poll for up to 5 minutes.
  - If tx not re-confirmed after 5 minutes → order `failed`, manual review triggered.
  - System must check delivery status BEFORE deciding re-org handling path.
- **BR-024:** **Nonce conflict:** If tx is replaced (same nonce, higher gas) → system detects new tx hash from same wallet, same amount. If new tx confirmed → order `confirmed`.
- **BR-025:** **Dropped transaction:** If tx disappears from mempool without confirmation → order `timeout` → move to `failed` after background monitoring window.
- **BR-026:** **Gas spike:** 
  - Gas fee is always paid by the user (standard Web3 behavior — user pays from wallet)
  - System does NOT refund gas difference under any circumstance
  - If actual gas exceeds estimate by >20% → log warning + flag for next estimate recalibration
  - Next estimate adds +25% buffer (capped at 2x original estimate — no unbounded buffer growth)
  - System does NOT auto-adjust estimate mid-transaction after user has confirmed
  - If gas spike causes tx to fail (user's wallet rejects due to insufficient gas) → order `failed`, user may retry with new gas estimate
- **BR-027:** **Chain RPC down:** If RPC endpoint returns error → retry with backup RPC (max 3 backup nodes). If all fail → show "Network temporarily unavailable. Your payment status will be updated automatically."
- **BR-028:** **Pending > 45s then confirmed:** Handled by BR-022 (background monitoring). User receives notification.

### 5.5 Duplicate & Idempotency
- **BR-029:** Stripe: all Payment Intent calls use idempotency key = `{order_id}_{attempt_number}`. Same key = same result, no double charge.
- **BR-030:** If user clicks "Pay" 3 times rapidly → system debounces: only 1 Payment Intent created per order per 5 seconds.
- **BR-031:** Web3 duplicate detection — a transaction is considered duplicate if ALL of the following match within 60 seconds:
  - Same wallet address
  - Same recipient address
  - Same token
  - Same amount (pre-rounding)
  - No new order created (same order_id)
  - **Note:** User buying 2 different items with same price = different order_id = NOT duplicate. Only block if all 5 conditions match.
- **BR-032:** If frontend retries order creation (network retry) → system returns existing order via idempotency key, does not create duplicate.

### 5.6 Stripe-Specific
- **BR-033:** If Stripe returns `card_declined` → show "Your card was declined. Please try another payment method."
- **BR-034:** If Stripe returns `insufficient_funds` → show "Insufficient funds on your card."
- **BR-035:** If Stripe returns `expired_card` → show "Your card has expired."
- **BR-036:** 3D Secure: if `requires_action` → redirect to bank auth page. If auth succeeds → `confirmed`. If auth fails or times out → `failed`.
- **BR-037:** Stripe webhook must be verified using Stripe signature (`stripe-signature` header). Unverified webhooks must be rejected with HTTP 400.

### 5.7 Refund & Chargeback
- **BR-038:** Stripe refund: initiated via admin panel or API. Full refund or partial refund supported. Order moves to `refund_pending` → `refunded` or `partially_refunded`.
- **BR-039:** Crypto refund: manual only (system cannot reverse on-chain tx). QA team initiates manual transfer. Order marked `refunded` only after manual confirmation.
- **BR-040:** Partial refund: supported for Stripe only. For crypto, partial refund requires manual approval from finance team.
- **BR-041:** Chargeback: if bank files chargeback → order moves to `chargebacked`. System flags user account for review. No automatic refund processing.
- **BR-042:** Refund window: Stripe refunds allowed within 90 days of original charge.

### 5.8 Network Mismatch
- **BR-043:** If wallet is on wrong network → show "Wrong network. Please switch to [required network] in your wallet." Block transaction until network is switched.

### 5.9 Gas Fee
- **BR-044:** Gas fee must be estimated and displayed BEFORE user confirms transaction.
- **BR-045:** If gas estimation fails → show "Unable to estimate gas fee. Proceed with caution." Allow user to continue.
- **BR-046:** Gas estimate adds 10% buffer by default to reduce failure risk.

### 5.10 Fraud & Abuse Prevention
- **BR-047:** Rate limiting: max 10 payment attempts per user per hour. On exceed → HTTP 429, show "Too many payment attempts. Try again in [X] minutes."
- **BR-048:** Bot/replay attack: all payment requests must include signed JWT token. Token expires in 15 minutes. Replayed tokens rejected with HTTP 401.
- **BR-067:** JWT expiry vs rate lock conflict — if user waits 14+ minutes:
  - Rate lock expires at 3 minutes → system auto-refreshes rate silently if user is still on page
  - JWT expires at 15 minutes → system must prompt re-authentication BEFORE showing updated rate
  - Flow: JWT expired → force re-login → re-fetch rate → show new amount → user confirms
  - If JWT expires during active `processing` state → do NOT interrupt. Complete current payment flow, handle auth on next action.
- **BR-049:** Amount tampering: payment amount must be validated server-side against order total. If frontend sends modified amount → reject with HTTP 400 "Payment amount mismatch."
- **BR-050:** Recipient address tampering: recipient address is set server-side only. Frontend cannot override recipient address.
- **BR-051:** Signature verification: all Web3 transaction parameters (amount, recipient, network) must be signed by backend. Frontend submits signed payload to wallet — cannot modify parameters.

---

## 6. Webhook & Async Handling

### 6.1 Stripe Webhook Events

| Event | Action |
|-------|--------|
| `payment_intent.succeeded` | Order → `confirmed`. Send confirmation email. |
| `payment_intent.payment_failed` | Order → `failed`. Send failure notification. |
| `payment_intent.requires_action` | Trigger 3D Secure redirect. |
| `charge.refunded` | Order → `refunded` or `partially_refunded`. Send refund confirmation. |
| `charge.dispute.created` | Order → `chargebacked`. Flag user account. |

### 6.2 Webhook Processing Rules
- **BR-052:** Each webhook must be processed within 5 seconds of receipt.
- **BR-053:** Webhook endpoint must return HTTP 200 immediately, process async.
- **BR-054:** If webhook processing fails → retry up to 3 times with exponential backoff (5s, 25s, 125s).
- **BR-055:** Duplicate webhook events (same event ID) must be deduplicated — processed only once.
- **BR-056:** Webhook reconciliation: every hour, system cross-checks orders in `processing` state > 10 minutes against Stripe API and blockchain. Auto-update stale orders.

### 6.3 Web3 Async Handling
- Active polling: 3s interval, 15 retries (45s)
- Background monitoring: after timeout, check every 30s for 10 minutes
- Late confirmation handler: if tx confirmed after timeout → BR-022 applies

---

## 7. Currency Conversion Rules

### 7.1 Exchange Rate Source
- **Primary:** CoinGecko API (real-time, refreshed every 60 seconds)
- **Fallback:** CoinMarketCap API (if CoinGecko unavailable)
- **Emergency fallback:** Last cached rate (max age: 5 minutes). If cache > 5 minutes old → block crypto payment, show "Exchange rate unavailable. Please use card payment."

### 7.2 Rounding & Precision
| Token | Decimals | Rounding Strategy |
|-------|----------|-------------------|
| ETH | 18 | Round up to 8 significant decimals |
| MATIC | 18 | Round up to 6 significant decimals |
| SOL | 9 | Round up to 6 significant decimals |
| USDC (ETH) | 6 | Round up to 2 significant decimals |
| USDC (SOL) | 6 | Round up to 2 significant decimals |
| BNB | 18 | Round up to 6 significant decimals |
| ARB | 18 | Round up to 6 significant decimals |

- Always round **up** (ceiling) to prevent underpayment
- Display rate and calculated amount to user before confirmation

### 7.3 Slippage Tolerance & Rounding Order of Operations

**Sequence must follow this exact order:**
1. **Quote amount** — fetch real-time rate, calculate raw token amount
2. **Lock rate** — rate locked for 3 minutes
3. **Apply slippage check** — verify actual on-chain amount is within ±2% of quoted amount
4. **THEN apply rounding** — round up AFTER slippage check to prevent over-collect

**Rules:**
- **BR-057:** Slippage tolerance: ±2% from quoted rate at time of transaction creation
- **BR-058:** If actual on-chain amount is within ±2% of expected (pre-rounding) → accept as valid
- **BR-059:** If actual on-chain amount is <98% of expected (pre-rounding) → order `failed`, show "Payment amount insufficient due to price movement. Please retry."
- **BR-060:** Rate is locked for 3 minutes after user initiates payment. If user takes >3 minutes → recalculate rate and show updated amount before allowing confirmation.
- **BR-066:** Rounding is applied to the final display amount AFTER slippage validation. Rounding up is for user-facing display only — slippage check uses pre-rounded value to prevent over-collect.

---

## 8. Input Constraints

| Field | Type | Constraints |
|-------|------|------------|
| Payment Amount (USD) | Decimal | Min: $1.00, Max: $10,000.00, 2 decimal places |
| Card Number | String | 16 digits, Luhn algorithm valid |
| Card Expiry | String | MM/YY format, must not be expired |
| CVV | String | 3-4 digits, never stored |
| Wallet Address (ETH/Rabby) | String | 42 chars, starts with `0x`, EIP-55 checksum valid |
| Wallet Address (Solana/Phantom) | String | 32-44 chars, Base58 encoded |
| Transaction Hash (ETH) | String | 66 chars: `0x` + 64 hex characters |
| Transaction Hash (Solana) | String | 87-88 chars, Base58 encoded |
| Network | Enum | `ethereum`, `polygon`, `solana`, `bsc`, `arbitrum` |
| Idempotency Key | String | `{order_id}_{attempt_number}`, max 64 chars |

---

## 9. Security Requirements

- Card CVV must never be stored or logged
- Wallet private keys must never be requested under any circumstance
- All Stripe API calls use server-side secret key only — never exposed to frontend
- Transaction signing occurs in user's wallet only — backend never holds signing keys
- All payment endpoints require valid JWT (expires 15 minutes)
- Stripe webhook signature verified via `stripe-signature` header
- Amount and recipient address set server-side, signed — frontend cannot tamper
- Rate limit: 10 payment attempts per user per hour

---

## 10. Audit & Logging

Every payment event must log:

| Field | Description |
|-------|-------------|
| `timestamp` | ISO 8601 UTC |
| `user_id` | Internal user identifier |
| `order_id` | Unique order identifier |
| `ip_address` | User IP at time of payment |
| `user_agent` | Browser/client user agent |
| `payment_method` | stripe / metamask / phantom / rabby |
| `network` | blockchain network (if Web3) |
| `wallet_address` | Wallet address (if Web3) |
| `card_last4` | Last 4 digits (if Stripe) |
| `amount_usd` | Amount in USD |
| `amount_token` | Amount in crypto + token symbol (if Web3) |
| `exchange_rate` | Rate used at time of transaction |
| `status` | Order state at time of event |
| `error_code` | Error code if failed |
| `tx_hash` | On-chain transaction hash (if Web3) |
| `block_number` | Block number of confirmation (if Web3) |
| `webhook_event_id` | Stripe webhook event ID (if applicable) |
| `attempt_number` | Retry attempt number |

---

## 11. Error States & Messages

| Error Code | Trigger | Message Shown to User |
|------------|---------|----------------------|
| `card_declined` | Stripe decline | "Your card was declined. Please try another payment method." |
| `insufficient_funds` | Stripe / wallet balance | "Insufficient funds." |
| `expired_card` | Card expired | "Your card has expired." |
| `wallet_rejected` | User rejects tx in wallet | "Transaction rejected by user." |
| `connection_rejected` | User rejects wallet connect | "Connection rejected. Please try again." |
| `wrong_network` | Wallet on wrong chain | "Wrong network. Please switch to [network]." |
| `timeout` | No on-chain confirmation in 45s | "Transaction timed out. Check your wallet for status." |
| `duplicate_tx` | Same tx within 60s | "Duplicate transaction detected." |
| `invalid_address` | Malformed wallet address | "Invalid wallet address." |
| `gas_estimation_failed` | RPC error | "Unable to estimate gas. Proceed with caution." |
| `rpc_down` | All RPC nodes fail | "Network temporarily unavailable. Your payment status will be updated automatically." |
| `rate_limit` | >10 attempts/hour | "Too many payment attempts. Try again in [X] minutes." |
| `amount_mismatch` | Tampered amount | "Payment amount mismatch. Please refresh and retry." |
| `rate_expired` | User takes >3 min | "Exchange rate has been updated. New amount: [X token]." |
| `reorg_detected` | Chain re-org | "Network reorg detected. Your payment is being re-verified." |
| `slippage_exceeded` | Amount <98% expected | "Payment amount insufficient due to price movement. Please retry." |
| `wallet_disconnected` | Wallet disconnects mid-flow | "Wallet disconnected. Please reconnect and try again." |

---

## 12. UX Recovery Rules

- **BR-061:** User refreshes page during `processing` → show "Your payment is being processed. Please wait." System continues polling in background.
- **BR-062:** User closes browser during `processing` → order remains `processing`. On next visit to order page, system checks latest status and updates UI.
- **BR-063:** User navigates to success URL manually (bookmarked) → system validates order status server-side. If not `confirmed` → redirect to order status page.
- **BR-064:** User opens 2 tabs with same order → second tab blocked with "Payment already in progress for this order."
- **BR-065:** User goes back after successful payment → system detects `confirmed` order, shows "This order has already been paid." Does not re-initiate payment.

---

## 13. Non-Functional Requirements

- Payment flow must complete (or fail gracefully) within 60 seconds end-to-end
- Stripe webhook must be processed within 5 seconds of receipt
- System must handle **100 concurrent payment sessions per region** (not system-wide)
- TPS expectation: 10 transactions per second under normal load, 50 TPS during load spikes
- Load spike behavior: queue excess requests, maintain response time < 3 seconds for 95th percentile
- Graceful degradation: if crypto payment service is down → disable Web3 payment option, show "Crypto payment temporarily unavailable. Please use card."
- Transaction polling interval: 3 seconds, max 15 retries (45 seconds total)
- Background monitoring: 30-second interval for 10 minutes after timeout

---

## 14. Advanced Security & Double-Spend Prevention

### 14.1 Double-Spend Prevention on Backend

Backend must verify ALL of the following before marking order `confirmed`:

- **BR-068:** Tx sender address matches `wallet_address` stored on order — reject if mismatch
- **BR-069:** Tx value matches expected token + amount (using pre-rounding value within slippage tolerance) — reject if mismatch
- **BR-070:** Tx recipient address matches system's designated recipient address — reject if mismatch
- **BR-071:** Signed payload from backend matches tx parameters — reject if signature invalid
- **BR-072:** Tx hash has not been used for any other order (global tx hash uniqueness check) — reject if tx hash already associated with another order

### 14.2 Frontend Integrity

- **BR-073:** All transaction parameters (amount, recipient, token, network) are generated and signed server-side. Frontend receives signed payload and submits to wallet — cannot modify any parameter.
- **BR-074:** If frontend sends modified amount or recipient → backend rejects with HTTP 400 "Invalid signed payload."

### 14.3 Audit Trail Integrity

- **BR-075:** All audit log entries are immutable — no update or delete allowed after write.
- **BR-076:** Log entries must include `block_number` and `tx_hash` for all Web3 events to support on-chain reconciliation.
- **BR-077:** Webhook event IDs must be stored to support idempotent webhook processing and audit replay.

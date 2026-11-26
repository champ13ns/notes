# Day 6 — Digital Banking & Payment Systems

**IBPS RRB Scale II IT Officer | Financial Awareness**  
**Notes updated: 26 November 2026**

## 1. Digital payments — the big picture

A digital payment moves value electronically rather than by physical cash. A payment ecosystem may include:
1. Customer / payer
2. Payer's bank
3. Payment app or interface
4. Payment network / switch
5. Beneficiary's bank
6. Merchant or payee
7. Clearing, settlement and reconciliation processes

**Banking exam lens:** Know the operator, purpose, speed, limits, minimum amount, availability and use case of each system.

---

## 2. RBI vs NPCI vs NBBL

| Institution | Role | Systems / examples |
|---|---|---|
| RBI | Central bank, regulator and operator of key payment systems | NEFT, RTGS; regulates payment systems |
| NPCI | Retail payment infrastructure organisation | UPI, IMPS, RuPay, AePS, NACH |
| NBBL | NPCI subsidiary operating Bharat Bill Payment System ecosystem | BBPS / Bharat Connect |

**Trap:** UPI is not operated by RBI; it is NPCI's payment platform. NEFT and RTGS are RBI-operated systems.

---

## 3. UPI — Unified Payments Interface

- NPCI-developed instant payment interface.
- Launched in **2016**.
- Enables payments using UPI ID/VPA, mobile-linked identity, QR, account details and supported modes.
- Supports P2P and P2M (person-to-merchant) payments.
- Works across participating banks and apps; it is interoperable.
- Payment app does not itself hold the customer's bank deposit merely because it displays the account.

### Typical flow
Customer initiates payment → app authenticates request → payer bank validates/authenticates → NPCI routing/switch coordinates → beneficiary bank processes credit → status response/reconciliation.

### UPI limit — exam note
Standard per-transaction limit is commonly **₹1 lakh**. Higher limits apply for specified verified merchant categories/transactions, with category-specific caps (often up to ₹5 lakh); banks/apps may impose lower customer-level limits and risk controls.

Do not treat one headline limit as applicable to every UPI payment.

### UPI PIN
- PIN authorises debit/payment.
- **You do not enter a UPI PIN to receive money.**
- Never share PIN, OTP, screen-sharing access or remote-control access with callers.

### UPI AutoPay
Used for recurring mandates (subscriptions, bills, EMIs, etc.). Mandate limits and additional authentication rules depend on the type/category and current NPCI/RBI rules.

---

## 4. UPI Lite

A wallet-like on-device/offline-friendly small-payment feature within supported UPI apps; transactions are designed to reduce the need for full bank-account interaction for every small payment.

Enhanced commonly tested limits (introduced in Dec 2024):
- Maximum per transaction: **₹1,000**
- Maximum wallet balance: **₹5,000**

Limits/features can vary with the latest regulatory/NPCI implementation.

**UPI Lite X** has offline capabilities for eligible devices/conditions; do not assume all UPI Lite payments are completely offline.

---

## 5. NEFT — National Electronic Funds Transfer

- Operated by **RBI**.
- Available **24×7**, including holidays, since 16 December 2019.
- Settles in half-hourly batches (48 batches in a day under the standard schedule).
- RBI does not prescribe a minimum/maximum transfer amount for NEFT; banks may set customer-level limits.
- Suitable for routine account-to-account transfers.

**NEFT = batch settlement in half-hourly cycles**, not individual real-time gross settlement.

---

## 6. RTGS — Real Time Gross Settlement

- Operated by **RBI**.
- Available 24×7 since **14 December 2020**.
- Minimum transaction: **₹2 lakh**.
- No RBI-prescribed upper ceiling; bank/customer limits may apply.
- “Real Time” = transaction processing/settlement individually as it is processed.
- “Gross” = each transaction settled individually, not netted with others.

### RTGS vs NEFT

| Feature | NEFT | RTGS |
|---|---|---|
| Operator | RBI | RBI |
| Settlement | Half-hourly batches | Real-time, gross |
| Minimum | No RBI minimum | ₹2 lakh |
| Upper ceiling | No RBI ceiling | No RBI ceiling |
| Use case | Routine transfers | Large-value urgent transfers |

---

## 7. IMPS — Immediate Payment Service

- Operated by **NPCI**.
- Instant interbank funds transfer.
- Available 24×7.
- Common maximum per transaction: **₹5 lakh** (some modes/categories and banks may have separate limits).
- Suitable for instant transfer when UPI is not used/available or customer prefers account/mobile-based IMPS.

**Trap:** IMPS is NPCI; NEFT and RTGS are RBI.

---

## 8. AePS — Aadhaar Enabled Payment System

NPCI-operated interoperable system enabling Aadhaar-authenticated banking transactions at Micro-ATMs / Business Correspondent touchpoints.

Typical services:
- Cash withdrawal
- Cash deposit (where supported)
- Balance enquiry
- Mini statement
- Aadhaar-based fund transfer (supported services)

Authentication is based on Aadhaar-linked bank account and biometric / approved authentication method.

**Financial inclusion angle:** enables basic banking access without requiring customer to visit a branch or use a smartphone.

---

## 9. BBPS / Bharat Connect

- BBPS = Bharat Bill Payment System.
- Operated by NPCI subsidiary **NBBL**.
- Bharat Connect is the customer-facing ecosystem/brand for interoperable bill payments.
- Supports billers and payment channels in an interoperable framework.

Common bill types: electricity, gas, water, telecom, DTH, FASTag and other supported billers.

**Trap:** Bharat Connect/BBPS is bill-payment infrastructure, not the same as UPI transfer.

---

## 10. NACH — National Automated Clearing House

NPCI's electronic system for recurring and bulk payments.

Use cases:
- Bulk credit: salary, pension, dividend, subsidy
- Bulk debit: EMI, insurance premium, SIP, utility payments

Mandate-based recurring payments allow an organisation to collect/pay at scheduled intervals under customer authorisation.

**NACH ≠ NEFT:** NACH is designed for bulk/recurring clearing; NEFT is a general funds-transfer system.

---

## 11. APBS — Aadhaar Payment Bridge System

APBS uses Aadhaar mapping to route eligible government benefits to the beneficiary's bank account.

**DBT (Direct Benefit Transfer)** is the policy/programme mechanism to transfer benefits directly; APBS is one payment-routing mechanism used for eligible Aadhaar-based DBT.

**Trap:** Aadhaar number is not itself a bank account. NPCI mapper routes according to the customer's active Aadhaar-bank mapping.

---

## 12. RuPay

- India's domestic card payment network developed by NPCI.
- Supports debit, credit and prepaid card products (depending on issuer/product).
- Used at ATM, POS and e-commerce acceptance points.
- Domestic network alternative alongside international card networks.

**Trap:** RuPay is a card network/brand; UPI is a payment interface; both are NPCI products but different functions.

---

## 13. CBDC — Central Bank Digital Currency / Digital Rupee (e₹)

- Digital form of sovereign currency issued by RBI.
- Liability of the central bank, unlike commercial-bank deposit money.
- Retail CBDC: public/customer use.
- Wholesale CBDC: institutional/financial-market use.
- Pilots began in India in 2022; use and pilot features evolve.

### CBDC vs UPI
| CBDC / e₹ | UPI |
|---|---|
| Digital sovereign money | Payment interface / system |
| Issued by RBI | NPCI platform |
| Represents digital currency | Moves funds held in bank accounts (typically) |
| Wallet may hold e₹ | UPI app links payment to account/payment instruments |

UPI is not itself money; it is a payment mechanism. CBDC is money in digital form.

---

## 14. Clearing, Settlement & Reconciliation

### Clearing
Process of transmitting, validating, calculating and preparing payment obligations between participants.

### Settlement
Final transfer of funds between participants to discharge payment obligations.

### Reconciliation
Comparing records across systems (bank ledger, payment switch, settlement report, merchant records) to identify mismatches.

**Example:** 100 payments of ₹100 each:
- Gross value = ₹10,000.
- If participants owe each other payments, net settlement calculates each participant's net obligation.
- RTGS is gross settlement; other arrangements may use net/batch settlement.

### Backend engineering relevance
- Use idempotency keys to avoid duplicate credit on retry.
- Store separate transaction states: initiated, pending, success, failed, reversed, settled.
- Keep immutable audit trails.
- Never assume timeout means failure; query status/reconcile before retrying blindly.
- Reconcile switch logs, bank ledger and settlement files.
- Use retry with backoff, but avoid duplicate financial postings.
- Maintain maker-checker for high-risk admin actions.

---

## 15. Failed transactions & compensation

RBI framework has prescribed reversal/compensation timelines for specified failed digital transactions. Commonly tested examples include:

| Scenario | Auto-reversal deadline (framework example) |
|---|---|
| UPI fund transfer: payer debited but beneficiary not credited | T+1 day |
| UPI merchant payment: debit but confirmation not received | T+5 days |
| IMPS: debit but beneficiary not credited | T+1 day |

Delayed reversal compensation may be **₹100 per day** beyond the applicable prescribed TAT for covered cases, subject to the framework and conditions.

**T = transaction date.** Exact applicability depends on transaction type, RBI framework and case facts.

**Exam trap:** UPI P2P and UPI merchant transactions can have different reversal TATs.

---

## 16. Digital payment security & fraud prevention

### Common frauds
- Phishing / fake links
- UPI collect-request scams
- Fake customer-care numbers
- Screen-sharing / remote-access scams
- SIM swap
- QR-code manipulation
- OTP/PIN theft
- Social engineering
- Malware and credential theft

### Customer safety
- UPI PIN is entered only to authorise payment/debit.
- Do not share PIN/OTP/password.
- Receiving money does not require PIN entry.
- Verify payee name and amount before approving.
- Do not install remote-access apps at a stranger's request.
- Report unauthorised transactions immediately to bank/payment app and official cybercrime reporting channels.

### IT officer controls
- Strong authentication and device binding
- Transaction velocity and risk limits
- Fraud detection rules / anomaly monitoring
- Encryption and secure key management
- API security, rate limiting and threat detection
- Secure logging (never log PIN/OTP)
- Incident response and customer notification
- Reconciliation and exception monitoring

---

## 17. Quick comparison table

| System | Operator | Key use | Availability / key limit |
|---|---|---|---|
| UPI | NPCI | Instant retail payment | 24×7; standard cap commonly ₹1 lakh, category exceptions |
| UPI Lite | NPCI ecosystem | Small payments | ₹1,000 txn; ₹5,000 wallet (enhanced limits) |
| NEFT | RBI | General transfers | 24×7; half-hourly batches; no RBI min/max |
| RTGS | RBI | Large urgent transfer | 24×7; min ₹2 lakh; no RBI max |
| IMPS | NPCI | Instant bank transfer | 24×7; common max ₹5 lakh |
| AePS | NPCI | Aadhaar-enabled basic banking | BC/Micro-ATM |
| BBPS/Bharat Connect | NBBL | Interoperable bill payments | Multiple billers |
| NACH | NPCI | Bulk/recurring payments | Mandate based |
| APBS | NPCI | Aadhaar-based DBT routing | Aadhaar mapper |
| RuPay | NPCI | Domestic card network | ATM/POS/e-commerce |
| CBDC/e₹ | RBI | Sovereign digital currency | Retail/wholesale pilots |

*Limits are indicative exam-revision values. Bank, transaction category, channel, risk controls and latest circulars can change actual limits.*

---

## 18. Practice MCQs

**Q1.** UPI is operated by:  
A. RBI  B. NPCI  C. SEBI  D. NABARD

**Q2.** RTGS minimum amount is:  
A. ₹10,000  B. ₹1 lakh  C. ₹2 lakh  D. ₹5 lakh

**Q3.** NEFT settlement is generally:  
A. Only monthly  B. Half-hourly batches  C. Only real-time gross  D. Cash settlement

**Q4.** Which payment system is RBI-operated?  
A. IMPS  B. RuPay  C. NEFT  D. AePS

**Q5.** IMPS is associated with:  
A. NPCI  B. SEBI  C. IRDAI  D. NABARD

**Q6.** AePS primarily enables:  
A. Bond trading  B. Aadhaar-enabled basic banking  C. Forex settlement  D. Credit rating

**Q7.** Bharat Connect / BBPS ecosystem operator is:  
A. RBI  B. NBBL  C. SEBI  D. SBI

**Q8.** NACH is especially useful for:  
A. One-time large RTGS transfer  B. Bulk/recurring payments  C. Stock exchange settlement  D. Cash withdrawal only

**Q9.** CBDC/e₹ is:  
A. A private cryptocurrency  B. Digital sovereign currency issued by RBI  C. A UPI ID  D. A debit card

**Q10.** UPI PIN is required to:  
A. Receive every payment  B. Authorise a debit/payment  C. Check every QR code  D. Create an Aadhaar number

### Answer key
1. **B** — NPCI operates UPI.  
2. **C** — RTGS minimum ₹2 lakh.  
3. **B** — NEFT settles in half-hourly batches.  
4. **C** — NEFT is RBI-operated.  
5. **A** — IMPS is NPCI.  
6. **B** — Aadhaar-enabled basic banking.  
7. **B** — NBBL operates BBPS/Bharat Connect.  
8. **B** — Bulk and recurring payments.  
9. **B** — Digital sovereign currency issued by RBI.  
10. **B** — PIN authorises debit; it is not needed to receive money.

---

## Official reference reading
- RBI payment systems: https://www.rbi.org.in/
- NPCI product overview: https://www.npci.org.in/
- UPI: https://www.npci.org.in/what-we-do/upi/product-overview
- NEFT: https://www.rbi.org.in/
- RTGS: https://www.rbi.org.in/
- IMPS: https://www.npci.org.in/what-we-do/imps/product-overview
- AePS: https://www.npci.org.in/what-we-do/aeps/product-overview
- Bharat Connect / BBPS: https://www.npci.org.in/what-we-do/bharat-billpay/product-overview
- NACH: https://www.npci.org.in/what-we-do/nach/product-overview
- RBI failed transaction TAT / compensation framework: https://www.rbi.org.in/

**Day 6 complete.** Prioritise operator mapping, NEFT vs RTGS, UPI/IMPS limits, BBPS/NACH/APBS, CBDC vs UPI and failed transaction TAT.

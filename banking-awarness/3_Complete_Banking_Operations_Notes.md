# IBPS RRB Scale II — IT Officer
## Financial Awareness | Day 3: Banking Operations, Deposits, Loans & NPAs

**Revision edition: 2 November 2026**  
**Format:** exam-oriented explanations, formulas, examples, traps, and 25 MCQs.  
**Scope:** the Day 3 topics studied together; Day 4 covers recovery laws such as SARFAESI, DRT and IBC.

> **Study priority:** Understand deposits as bank liabilities, loans as bank assets, CASA, loans and working capital, SMA / NPA rules, agricultural exceptions, and deposit insurance. All interest rates used in numericals are **illustrative**, not quotes from a particular bank.

---

## 1. How a commercial bank works

A bank accepts deposits and provides credit and payment services, while maintaining regulatory reserves, liquidity, and capital.

**From the bank's point of view:**
- Customer deposit = **liability**: the bank owes money to the depositor.
- Loan to a customer = **asset**: borrower owes money to the bank.
- Cash, RBI balances and investments = other assets.
- Equity capital and reserves = owners' funding, distinct from customer deposits.

**Example:** Ramesh deposits ₹1 lakh in SBI. SBI records a ₹1 lakh deposit liability; the deposit may increase its cash/reserve assets. If SBI advances a ₹1 lakh loan, the loan is SBI's asset. This is simplified: a bank must also manage reserve requirements, liquidity, credit risk and capital; it cannot lend all deposits without constraint.

**Bank income:** interest on loans and investments, fees and other income. **Bank expenses:** deposit interest, salaries, IT systems, credit provisions and funding expenses. Net interest income (NII) is interest earned **minus** interest paid. **Net interest margin (NIM)** compares NII with average earning assets and is not the same as NII or final profit.

---

## 2. Demand deposits vs time deposits

**Demand deposits:** ordinarily withdrawable on demand, subject to account rules. Savings and current accounts are typical examples.

**Time/term deposits:** kept for an agreed period. Fixed Deposit (FD) and Recurring Deposit (RD) are examples; some products permit premature withdrawal with conditions or penalties, while some are non-callable.

| Feature | Savings account | Current account | Fixed deposit (FD) | Recurring deposit (RD) |
|---|---|---|---|---|
| Primary user/use | Personal savings, day-to-day banking | Businesses, high-frequency transactions | Lump sum for chosen tenure | Regular periodic savings |
| Deposit type | Demand | Demand | Time | Time |
| Interest | Usually yes | Generally no | As per agreed deposit terms | As per agreed deposit terms |
| Typical deposits | Flexible | Flexible | Initial lump sum | Scheduled instalments |
| Withdrawal | Account rules | Account rules | Maturity / premature conditions | Maturity / premature conditions |

**Savings account interest:** Under RBI deposit directions, calculated on the **daily product / daily closing balance**. Bank determines its applicable rate under the rules.

**Numerical:** ₹1,00,000 remains for 30 days at an **assumed** 3% p.a.:

`Interest ≈ 100000 × 0.03 × 30/365 = ₹246.58`.

**Current account:** As a general rule, no interest is paid on domestic current account deposits; a narrow deceased-depositor exception exists.

**FD example:** ₹1 lakh at 7% p.a. for one year gives ₹7,000 under **simple-interest assumption**. Actual FD interest depends on compounding frequency, tenure and payout instructions.

**RD example:** ₹5,000 deposited monthly for 12 months means **principal contributions = ₹60,000**. Each instalment has a different time invested; do **not** compute RD interest as though ₹60,000 were deposited at the start of the year.

**Exam traps:**
- Current account ≠ always an overdraft. Overdraft is a **separately sanctioned credit facility**.
- FD interest rate does not apply universally to every bank or tenure.
- Savings account = demand deposit; FD/RD = time deposits.

---

## 3. CASA ratio — low-cost funding

**CASA** stands for **Current Account + Savings Account** deposits.

`CASA Ratio (%) = [(Current account deposits + Savings account deposits) / Total deposits] × 100`

Example:
- Current deposits = ₹200 crore.
- Savings deposits = ₹300 crore.
- Term and other deposits = ₹500 crore.
- Total deposits = ₹1,000 crore.
- CASA ratio = `(200 + 300) / 1000 × 100 = 50%`.

Why banks like CASA: current deposits normally cost zero direct interest and savings deposits often cost less than term deposits. Higher CASA can help reduce funding cost, **other things being equal**. It does not guarantee greater profitability; account servicing costs, credit quality and loan yields matter too.

**NII vs profit example:** If bank earns ₹1 crore loan interest and pays ₹60 lakh interest on deposits, NII is ₹40 lakh. Salaries, technology, credit costs and taxes still need to be considered before arriving at net profit.

---

## 4. Loan products and vocabulary

| Term | Meaning |
|---|---|
| Principal | Amount borrowed |
| Interest | Cost of borrowing |
| Tenure | Period for repayment |
| EMI | Equated Monthly Instalment, generally includes principal and interest |
| Collateral | Asset/property providing security for a loan |
| Sanctioned limit | Approved maximum credit facility |
| Drawing power | Amount currently permitted under asset-backed working capital terms |
| Outstanding | Amount presently owed |
| Overdue | Amount not paid by its contractual due date |

### Secured vs unsecured credit
- **Secured:** credit backed by collateral/security (home loan mortgage, gold loan pledge, vehicle hypothecation).
- **Unsecured:** no specific collateral requirement (many personal loans); borrower still has contractual and legal repayment duties.

**Mortgage:** security interest in immovable property. **Pledge:** possession of movable collateral, such as pledged gold, generally with lender. **Hypothecation:** charge on movable assets while possession generally stays with borrower (e.g., financed vehicle, inventory).

### Term loan, CC and OD

**Term loan:** funding for defined purpose/asset; agreed disbursement and repayment schedule. Example: ₹6 lakh tractor loan repaid through instalments.

**Cash Credit (CC):** revolving working-capital finance, often secured by inventory/receivables and subject to drawing power. Example: wholesaler sanctioned ₹10 lakh CC limit to finance stock.

**Overdraft (OD):** bank allows eligible account to be overdrawn up to approved limit, for specified period and conditions. Example: account has ₹20,000 and eligible OD ₹2 lakh; customer may draw beyond account balance up to available approved OD.

In CC/OD, interest typically relates to actual utilised balances as per terms, not automatically the full sanctioned credit limit. **CC, OD and term loan are not synonyms.**

### Loan lifecycle (IT Officer lens)
1. Application and consent; collect KYC/credit information.
2. Underwriting, risk assessment, eligibility and collateral valuation.
3. Sanction with interest/repayment conditions.
4. Disbursement and core banking ledger postings.
5. Interest accrual, repayment schedules, reminders, reconciliation.
6. Detect overdue stress; SMA / NPA tagging and collection workflow.
7. Resolution, recovery or closure under applicable norms.

**Production-system point:** Financial computations require consistent day-end dates, transaction audit trails, idempotent payment posting and exact amount rounding rules. Incorrect day-end posting can cause wrong SMA/NPA classification.

---

## 5. Loan stress: SMA vs NPA

**SMA = Special Mention Account.** Indicates incipient loan stress before standard term-loan NPA threshold.

For standard non-agricultural **non-revolving** facilities:

| Tag | Overdue period |
|---|---|
| SMA-0 | Up to 30 days (after due date) |
| SMA-1 | More than 30 up to 60 days |
| SMA-2 | More than 60 up to 90 days |
| NPA | More than 90 days overdue (ordinary term-loan rule) |

If ₹12,000 EMI due on 1 Jan is unpaid, the ageing advances according to applicable date and bank's end-of-day process. Missing a single EMI does **not** instantly mean NPA.

**NPA = Non-Performing Asset.** An asset that no longer satisfies RBI income/repayment performance tests. **NPA ≠ loan cancellation**. Loan can still be recovered.

**Important special cases:**
- **CC/OD** can become NPA when deemed **out of order** under rules (e.g. continuously over sanctioned limit/drawing power, or no adequate credits during prescribed 90-day periods).
- For qualifying **short-duration crop loans**, overdue for **two crop seasons**; for qualifying **long-duration crop loans**, overdue for **one crop season**. A 'crop season' is determined for the relevant crop by SLBC. Not every agricultural-purpose loan automatically qualifies for these special conditions.
- Classification is normally based on the **borrower's account position**, with additional institution/facility-specific prudential rules. The authoritative RBI directions for the relevant bank category control.

### Why NPA harms bank
Interest income recognition becomes restricted. Bank must create provisions based on asset quality and security. More NPAs impair profitability, capital and credit availability.

---

## 6. Asset classification and provisioning

| Classification | Meaning |
|---|---|
| Standard asset | Performing, not classified NPA |
| Substandard asset | NPA for up to 12 months (general banking framework) |
| Doubtful asset | Has remained substandard for 12 months |
| Loss asset | Loss identified by bank/auditor/regulator but not fully written off |

**Provisioning:** recognising an expense / allowance for potential credit loss in accordance with applicable prudential and accounting norms.

**Gross NPA ratio:**

`Gross NPA ratio (%) = Gross NPAs / Gross advances × 100`

Example: Gross advances ₹10,000 crore; Gross NPAs ₹500 crore; GNPA ratio = **5%**.

**Net NPAs:** Gross NPAs after specified provisions and other adjustments. **Net NPA ratio's denominator also has regulatory adjustments** — do not simply subtract provisioning percentage from GNPA ratio.

**PCR (Provision Coverage Ratio):** Broadly indicates how much of gross NPAs is covered by provisions, using the applicable reporting definition.

**Write-off vs waiver:**
- **Write-off:** accounting removal / adjustment of asset; recovery rights can continue.
- **Waiver:** legal release of borrower obligation under authorised scheme/settlement.
- **Provision:** allowance for loss while loan exposure still exists.

Exam trap: A 'loss asset' is not automatically a loan waiver.

---

## 7. DICGC deposit insurance

**DICGC = Deposit Insurance and Credit Guarantee Corporation**, an RBI subsidiary providing statutory insurance for eligible deposits at insured banks.

**Core amount:** **₹5,00,000 per depositor per insured bank, in the same right and capacity**, covering **principal plus interest together**.

Example: Same insured bank, same depositor and ownership capacity:
- Savings ₹2 lakh + FD ₹4 lakh = ₹6 lakh aggregate; maximum standard insurance cover = **₹5 lakh**.
- Another branch of the **same bank** does **not** create a second ₹5 lakh limit.
- Deposits in a **different insured bank** have a separate eligible coverage limit.

Savings, current, fixed and recurring deposits may be covered subject to statutory exclusions. Insurance premium is paid by the **insured bank**, not directly by ordinary depositors.

**Exam trap:** ₹5 lakh is **per bank**, not ₹5 lakh per account or each branch. Includes accrued interest within ceiling.

---

## 8. One-page fast revision

| Keyword | Recall |
|---|---|
| Customer deposit on bank balance sheet | Liability |
| Customer loan on bank balance sheet | Asset |
| Savings / Current | Demand deposits |
| FD / RD | Term deposits |
| Current account interest | Generally zero |
| Savings interest method | Daily closing balance |
| CASA | (CA + SA) / Total deposits × 100 |
| FD vs RD | Lump-sum vs recurring instalments |
| Secured vs unsecured | With specified collateral vs without |
| Term loan | Scheduled disbursement/repayment |
| CC / OD | Revolving / overdraft credit facilities |
| SMA-0 | Up to 30 overdue days |
| SMA-1 | 31–60 overdue days |
| SMA-2 | 61–90 overdue days |
| Ordinary term-loan NPA | More than 90 days overdue |
| Crop short-duration NPA | Overdue for 2 crop seasons |
| Crop long-duration NPA | Overdue for 1 crop season |
| Substandard | NPA up to 12 months |
| Doubtful | Substandard for 12 months |
| Gross NPA ratio | Gross NPAs / Gross advances × 100 |
| DICGC | ₹5 lakh per depositor per insured bank, principal + interest |

---

## 9. Exam traps checklist

1. A customer's ₹1 lakh deposit is **not** the bank's asset — it is its liability.
2. CASA numerator excludes FDs and RDs.
3. High CASA **may** mean lower cost of funds; not automatic high profits.
4. A sanctioned OD limit is **not** a customer's savings balance.
5. Bank does not charge simple interest on the entire RD final sum for a full year.
6. One missed EMI ≠ immediate NPA.
7. SMA is **pre-NPA** warning classification.
8. CC/OD classification uses the **out-of-order** test rather than just EMI dates.
9. Crop-season exceptions do not cover every agricultural loan indiscriminately.
10. NPA, provision, write-off and waiver are different terms.
11. DICGC ₹5 lakh limit includes both principal and accrued interest.
12. Separate accounts or branches in same insured bank do not multiply cover.

---

## 10. Practice: 25 MCQs

Choose one correct answer, then compare with the answer key below.

1. A customer keeps ₹50,000 in a bank savings account. On the bank's books it is: (A) Asset (B) Liability (C) Equity (D) Profit.
2. A bank advances ₹5 lakh as personal loan. To the bank it is usually: (A) Asset (B) Liability (C) Deposit (D) Share capital.
3. Which is a demand deposit? (A) FD (B) RD (C) Savings (D) Five-year bond.
4. Interest on normal domestic current account deposits is generally: (A) 7% (B) 4% (C) None (D) Repo rate.
5. Savings interest is calculated on: (A) Monthly minimum always (B) Daily product/closing balance (C) Highest monthly balance (D) Yearly fixed balance.
6. ₹1,00,000 at 3% p.a. for 30 days (365-day basis) generates about: (A) ₹247 (B) ₹3,000 (C) ₹30 (D) ₹2,465.
7. A recurring deposit is characterised by: (A) One lump-sum only (B) Regular instalments (C) No tenure (D) No interest.
8. CA ₹120 Cr, SA ₹180 Cr, other deposits ₹700 Cr. CASA is: (A) 12% (B) 18% (C) 30% (D) 70%.
9. Usually a bank prefers more low-cost CASA because it can: (A) Cut funding cost (B) Avoid liquidity rules (C) Eliminate NPAs (D) Waive customer loans.
10. Which is the best example of a secured loan? (A) Unsecured personal loan (B) Gold loan secured by pledged gold (C) Credit card revolving balance (D) No-document loan.
11. A term loan commonly involves: (A) No repayment plan (B) Planned schedule of repayment (C) Free credit (D) Only deposit transactions.
12. A CC facility often finances: (A) Business working capital (B) Pension (C) Equity dividends (D) RBI currency printing.
13. OD allows an eligible customer to: (A) Borrow above account balance within approved limit (B) Receive free money (C) Avoid loan interest (D) Ignore sanctioned limits.
14. SMA expands to: (A) Standard Monetary Account (B) Special Mention Account (C) Statutory Margin Asset (D) Savings Management Account.
15. Ordinary loan 45 days overdue: (A) Standard with no stress (B) SMA-1 (C) SMA-2 (D) Doubtful.
16. Ordinary loan 75 days overdue: (A) SMA-0 (B) SMA-1 (C) SMA-2 (D) Loss asset automatically.
17. Usual term-loan NPA test is principal/interest overdue for: (A) 15 days (B) More than 90 days (C) 180 days in every case (D) 1 day.
18. A qualifying short-duration crop loan becomes NPA when overdue for: (A) 90 days always (B) Two crop seasons (C) One month (D) Five years.
19. A qualifying long-duration crop loan criterion is: (A) One crop season overdue (B) Two crop seasons (C) Every 30 days (D) No NPA norms.
20. A loan NPA for 8 months would ordinarily be classified: (A) Standard (B) Substandard (C) Doubtful (D) Deposit.
21. An NPA that has remained substandard for 12 months normally becomes: (A) Standard (B) Doubtful (C) Savings deposit (D) Repo.
22. Gross advances ₹2,000 Cr, Gross NPAs ₹100 Cr: GNPA ratio? (A) 2% (B) 5% (C) 10% (D) 50%.
23. Loan write-off always means borrower is legally freed? (A) Yes (B) No (C) Only for FDs (D) Only at RBI discretion.
24. Same insured bank: savings ₹3 lakh + FD ₹4 lakh in same right/capacity. Standard DICGC max: (A) ₹7 lakh (B) ₹10 lakh (C) ₹5 lakh (D) ₹3 lakh.
25. Two different insured banks: ₹5 lakh eligible deposits in each under same depositor. Under usual per-bank rule, potential cover is: (A) ₹5 lakh total (B) Up to ₹5 lakh at each bank (C) ₹2.5 lakh each (D) Not insured.

### Answer key, with brief explanations

| Q | Answer | Explanation |
|---|---|---|
| 1 | B | Deposits are payable to customers. |
| 2 | A | Loan receivable is an asset of the lender. |
| 3 | C | Savings is a demand deposit. |
| 4 | C | Domestic current accounts generally do not earn interest. |
| 5 | B | RBI prescribes daily-product basis. |
| 6 | A | ₹1,00,000 × 3% × 30/365 ≈ ₹246.58. |
| 7 | B | RD builds savings through instalments. |
| 8 | C | (120+180)/1000 = 30%. |
| 9 | A | CASA can lower average funding cost. |
| 10 | B | Pledged gold is collateral. |
| 11 | B | Term loans have repayment terms/schedule. |
| 12 | A | CC is commonly used for working capital. |
| 13 | A | OD is borrowing beyond balance within approval. |
| 14 | B | Special Mention Account. |
| 15 | B | 31–60 days = SMA-1. |
| 16 | C | 61–90 days = SMA-2. |
| 17 | B | Ordinary term-loan NPA: >90 days overdue. |
| 18 | B | Short-duration crops: two crop seasons. |
| 19 | A | Long-duration crops: one crop season. |
| 20 | B | NPA up to 12 months = substandard. |
| 21 | B | 12 months in substandard => doubtful. |
| 22 | B | 100/2000×100 = 5%. |
| 23 | B | Accounting write-off doesn't necessarily waive legal debt. |
| 24 | C | Same bank/capacity aggregate; cap ₹5 lakh. |
| 25 | B | Insurance per insured bank, subject to other terms. |

---

## 11. Source links for verifying rules

- [RBI: Commercial Banks – Interest Rate on Deposits Directions, 2025 (updated Oct 2026)](https://trackrbi.com/master-direction/13157/reserve-bank-of-india-commercial-banks-interest-rate-on-deposits-directions) — savings interest and current deposits.
- [RBI: Commercial Banks – Income Recognition, Asset Classification and Provisioning Directions, 2025 (updated July 2026)](https://trackrbi.com/notification/13146/reserve-bank-of-india-commercial-banks-income-recognition-asset-classification) — NPA/SMA classification, asset categories.
- [Ministry of Finance / DFS: Banking FAQs](https://www.financialservices.gov.in/banking) — SMA 0/1/2.
- [DICGC official FAQs](https://www.dicgc.org.in/FAQs) — ₹5 lakh cover, aggregation, exclusions.
- [RBI Handbook of Statistics / prudential norms](https://website.rbi.org.in/documents/d/rbi/handbookg27022025d0f3f53f5d3c4310a6bb2f8ac2175d3a) — bank-class-specific asset classification differences.

**Next lesson:** Day 4 — SARFAESI, DRT, Lok Adalat, IBC and real bank-loan recovery cases.

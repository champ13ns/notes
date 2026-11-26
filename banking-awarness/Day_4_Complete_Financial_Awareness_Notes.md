# Day 4 — NPA, Asset Classification, Provisioning & Recovery

**IBPS RRB Scale II IT Officer | Financial Awareness**  
**Notes updated: 26 November 2026**

> Exam lens: NPA definitions and classification, SMA stages, agricultural loan rules, provisioning, SARFAESI, DRT, ARC, IBC and recovery vocabulary.

---

## 1. What is an NPA?

A **Non-Performing Asset (NPA)** is a loan/advance or other credit exposure that has ceased to generate income for the bank because the borrower is not servicing principal and/or interest as required.

**Why it matters:** the bank cannot treat overdue interest as normal earned income, must classify the exposure correctly, make provisions for expected/identified loss and intensify recovery.

### Overdue vs NPA
- **Overdue:** an amount payable has not been paid by its due date.
- **NPA:** an exposure meets RBI's prudential classification criteria. Being overdue does not automatically mean that it is already an NPA; the overdue period/type of account matters.
- **Default:** failure to meet a payment or other obligation under the contract. A default may occur before the prudential NPA classification.

### The 90-day rule (ordinary loans)
A term loan generally becomes NPA when principal and/or interest remains overdue for **more than 90 days**. Similar 90-day norms apply to bills purchased/discounted and other specified receivables.

**Exam trap:** It is *more than 90 days*, not “one EMI missed” and not “90 days from sanction”.

---

## 2. Special NPA rules

| Exposure | NPA trigger |
|---|---|
| Term loan | Principal and/or interest overdue for more than 90 days |
| Bills purchased/discounted | Bill overdue for more than 90 days |
| OD/CC | Account remains “out of order” under RBI criteria |
| Short-duration crop loan | Overdue for **2 crop seasons** |
| Long-duration crop loan | Overdue for **1 crop season** |
| Other specified receivables | Amount remains overdue for more than 90 days |

### OD/CC: “Out of Order”
A Cash Credit (CC) / Overdraft (OD) account may be treated as out of order if, for the prescribed continuous 90-day period, any of these applies:

1. Outstanding continuously exceeds the sanctioned limit or drawing power.
2. Outstanding is within limit/DP, but there are no credits continuously for 90 days.
3. Credits during the period are insufficient to cover interest debited during that period.

**Remember:** CC/OD has a special “out of order” test; do not mechanically apply the term-loan EMI rule.

### Agricultural advances
- **Short-duration crops:** NPA if overdue for **two crop seasons**.
- **Long-duration crops:** NPA if overdue for **one crop season**.
- “Crop season” is determined for each crop by the State Level Bankers’ Committee (SLBC) in the State.
- The relaxed crop-season rule applies to eligible direct agricultural advances. Other agricultural-related lending may be governed by the ordinary 90-day rule.

**Mnemonic:** Short = 2 seasons; Long = 1 season.

---

## 3. SMA — Special Mention Account

SMA is an early-warning classification used to identify stress **before NPA**.

| Category | Overdue period |
|---|---:|
| SMA-0 | 1–30 days |
| SMA-1 | 31–60 days |
| SMA-2 | 61–90 days |
| NPA | More than 90 days / applicable NPA trigger |

Example: EMI due 1 January and still unpaid:
- 1–30 days overdue → SMA-0
- 31–60 days → SMA-1
- 61–90 days → SMA-2
- NPA classification when applicable threshold is crossed

Banks monitor SMA accounts closely and use them for collection, restructuring assessment and early intervention.

---

## 4. NPA Asset Categories

Once an asset becomes NPA, the bank classifies it further based on time spent in NPA status and recoverability.

| Category | Meaning |
|---|---|
| **Substandard** | NPA for up to 12 months; identifiable credit weaknesses |
| **Doubtful** | Has remained substandard for 12 months; recovery is highly uncertain |
| **Loss asset** | Loss has been identified by the bank, auditor or RBI inspection, but it has not been fully written off |

**Classification ladder:** Standard → SMA (stress warning) → NPA → Substandard → Doubtful → Loss.

**Trap:** “Doubtful” is not the same as 12 months overdue from the original EMI. It means the asset has remained in the *substandard category* for the prescribed 12-month period.

---

## 5. Borrower-level classification & upgradation

- RBI prudential rules generally require NPA classification at the **borrower level**: where one exposure to a borrower is NPA, other exposures of that borrower with the bank may also be classified as NPA, subject to the applicable rules.
- An NPA is not upgraded merely because the borrower pays a small amount or one EMI.
- Under the current conventional IRACP framework, upgradation requires repayment of **all arrears of principal and interest** as required by the applicable rules. Where a borrower has multiple credit facilities, arrears across the facilities must be cleared for upgrade.

**Exam trap:** A borrower can have multiple loans from one bank; classification/upgradation is not simply decided facility-by-facility.

---

## 6. Income Recognition — why NPA affects profit

For a performing/standard loan, interest is generally recognised on accrual as it becomes earned under applicable accounting/regulatory rules.

For an NPA:
- Stop recognising unrealised interest as normal accrued income.
- Interest income is generally recognised on a **cash/recovery basis**, as per applicable RBI rules.
- Reversal/adjustment of unrealised interest previously recognised may be required.
- Recovery of principal and interest must be tracked separately and appropriated according to policy/regulatory requirements.

**Backend analogy:** A scheduled EMI is not equivalent to a settled cash receipt. A bank should not report income merely because its system generated an interest accrual entry when the exposure is non-performing.

---

## 7. Provisioning — bank protects itself against credit loss

**Provision:** an expense/reserve recognised against expected or identified loss on a loan. It reduces profit and helps absorb losses.

**Write-off:** removal of an amount from the balance sheet after accounting treatment. It does **not automatically mean the borrower is forgiven or the bank stops recovery**.

### Common conventional IRACP provisioning figures (commercial banks)

These are exam-friendly general rates under the conventional framework; product, security, infrastructure and special rules may alter treatment.

| Asset category | General provision |
|---|---|
| Standard — direct Agriculture & SME | 0.25% |
| Standard — Commercial Real Estate (CRE) | 1.00% |
| Standard — CRE Residential Housing (CRE-RH) | 0.75% |
| Other standard loans | 0.40% |
| Substandard | 15% of outstanding |
| Substandard unsecured exposure | Additional 10% → generally 25% total |
| Doubtful — unsecured portion | 100% |
| Doubtful — secured portion, doubtful up to 1 year | 25% |
| Doubtful — secured portion, 1–3 years | 40% |
| Doubtful — secured portion, over 3 years | 100% |
| Loss asset | Write off; if retained in books, generally 100% provision |

### Example: doubtful loan
Outstanding = ₹10 lakh; realistic security value = ₹6 lakh.  
Unsecured portion = ₹4 lakh → provision at 100% = ₹4 lakh.  
Assume the secured portion has been doubtful for 2 years → 40% of ₹6 lakh = ₹2.4 lakh.  
**Illustrative total provision = ₹6.4 lakh.**

This is an illustration only: security valuation, eligible collateral and the exact RBI provision rules determine the actual amount.

### Provisioning Coverage Ratio (PCR)
PCR broadly indicates how much of gross NPAs has been covered by provisions.

**PCR = Provisions against NPAs ÷ Gross NPAs × 100**

Higher coverage generally means a larger cushion against recognised bad loans, but PCR alone does not tell the full story of a bank's asset quality.

---

## 8. Recovery mechanisms — the exam comparison

### A. SARFAESI Act, 2002
Full name: **Securitisation and Reconstruction of Financial Assets and Enforcement of Security Interest Act, 2002.**

Purpose: allows eligible secured creditors to enforce security interest and recover dues **without first filing a civil suit**, subject to statutory safeguards.

**Typical process:**
1. Borrower defaults; secured loan is classified as NPA under applicable rules.
2. Secured creditor issues a written demand under **Section 13(2)**, allowing **60 days** to pay.
3. Borrower may submit representation/objection; creditor must consider and communicate reasons for rejection within the statutory period.
4. If dues remain unpaid, creditor may take measures under **Section 13(4)**, such as possession of secured assets, taking over management in eligible cases, appointing a manager or sale/lease/assignment of the asset.
5. Assistance of the Chief Metropolitan Magistrate/District Magistrate may be sought under **Section 14** for possession.
6. Borrower/affected person may approach the DRT under **Section 17** within **45 days** of the relevant measure.
7. Appeal to DRAT under **Section 18** is generally within **30 days**, subject to statutory deposit conditions.

### SARFAESI limitations / exclusions — common MCQs
The Act does not apply to certain cases, including:
- Security interest created in **agricultural land**.
- Pledge of movables (covered by pledge law).
- Certain liens and specified aircraft/vessel interests.
- Security interest securing a financial asset **not exceeding ₹1 lakh**.
- Where amount due is **less than 20%** of principal and interest.

SARFAESI is primarily for **secured assets**; it is not a universal shortcut for every unsecured loan.

### B. DRT & DRAT
- **DRT:** Debts Recovery Tribunal; established under the Recovery of Debts and Bankruptcy Act, 1993 (earlier RDDBFI Act).
- **DRAT:** Debts Recovery Appellate Tribunal.
- DRT adjudicates recovery applications by banks/financial institutions and can issue a Recovery Certificate.
- The notified pecuniary threshold for DRT applications is **₹20 lakh** (notification dated 6 September 2018).

| Body | Role |
|---|---|
| DRT | Original recovery adjudication / borrower applications under applicable laws |
| DRAT | Appellate tribunal for DRT matters |

### C. ARC — Asset Reconstruction Company
An ARC acquires financial assets (often stressed/NPA loans) from banks and undertakes reconstruction/recovery.

- Bank transfers eligible stressed assets to ARC under a transaction.
- ARC may use recovery, restructuring, enforcement or other permitted resolution methods.
- ARC may issue Security Receipts (SRs) to eligible investors under the applicable framework.
- It helps a bank clean up its balance sheet and focus on fresh lending.

**ARC ≠ DRT:** ARC is a financial company dealing in asset reconstruction; DRT is a tribunal.

### D. Lok Adalat
Useful for compromise/settlement of eligible disputes, especially suitable cases. A settlement award has legal enforceability under the applicable framework.

### E. OTS — One-Time Settlement
A negotiated settlement where lender accepts agreed payment terms to close/resolve dues. It is a settlement decision, not an automatic borrower right.

### F. IBC — Insolvency and Bankruptcy Code, 2016
A structured insolvency-resolution/liquidation framework for eligible corporate debtors and other categories covered by the Code.

**Corporate CIRP basics:**
- Adjudicating authority: **NCLT**.
- Financial creditor application: Section 7.
- Operational creditor: Section 9, following demand notice/process.
- Corporate debtor: Section 10 (subject to law).
- Interim Resolution Professional (IRP) → Resolution Professional (RP).
- Committee of Creditors (CoC) primarily consists of financial creditors.
- Resolution plan requires the prescribed CoC voting threshold (commonly tested: **66%**).
- If resolution fails or legal conditions are met, liquidation may follow.

**Default threshold for corporate insolvency applications:** ₹1 crore, notified in March 2020. (Always check the current statutory position for any later notification.)

### IBC timeline — exam note
The commonly tested framework has a 180-day CIRP period, extension subject to law, and the overall statutory timeline is generally described as 330 days including litigation. Actual process/timelines may be affected by amendments and court decisions.

**2026 update:** The IBC (Amendment) Act, 2026 received Presidential assent on 6 April 2026; provisions were brought into force in phases. It updates timelines, creditor participation and resolution/liquidation processes. For current-affairs questions, distinguish the principal IBC 2016 from amendments and their commencement dates.

---

## 9. Write-off, waiver, compromise — do not mix these up

| Term | Meaning |
|---|---|
| Provision | Expense/cushion against possible or identified loss |
| Write-off | Accounting removal of loan/portion; recovery can continue |
| Waiver | Lender legally/contractually gives up recovery of a specified amount |
| OTS/compromise | Agreed settlement amount/terms with borrower |
| Restructuring | Changes repayment terms to address stress, subject to regulatory rules |

**Exam trap:** A bank write-off is **not automatically a loan waiver**.

---

## 10. Banking & IT Officer perspective

An NPA-management system should support:
- Reliable due-date and DPD (days-past-due) calculation.
- SMA/NPA classification rules that run automatically and consistently.
- Audit trails for classification overrides.
- Borrower-level aggregation across facilities.
- Interest accrual stop/reversal rules when status changes.
- Provisioning calculations and collateral valuation history.
- Recovery notices, payment tracking, legal case workflow and document storage.
- Reconciliation between loan ledger, GL, provision accounts and recovery receipts.
- Role-based access and immutable audit logs.

A manual spreadsheet that updates overdue status only at month-end can create regulatory and financial-reporting risk.

---

## 11. Last-minute revision table

| Topic | Must remember |
|---|---|
| Ordinary term loan NPA | >90 days overdue |
| SMA-0 / 1 / 2 | 1–30 / 31–60 / 61–90 days |
| Short crop | 2 crop seasons |
| Long crop | 1 crop season |
| Substandard | NPA for up to 12 months |
| Doubtful | 12 months in substandard category |
| Loss asset | Loss identified; not fully written off |
| SARFAESI demand notice | 60 days — Section 13(2) |
| SARFAESI measures | Section 13(4) |
| Magistrate assistance | Section 14 |
| DRT application under SARFAESI | Section 17; generally 45-day window |
| DRAT appeal | Section 18; generally 30 days |
| DRT pecuniary threshold | ₹20 lakh |
| IBC corporate default threshold | ₹1 crore |
| Provision on loss asset retained | 100% |
| Write-off | Does not automatically waive debt |

---

## 12. Practice MCQs

Try these before checking the key.

**Q1.** A term loan is generally classified as NPA when principal or interest is overdue for:  
A. 30 days  B. 60 days  C. More than 90 days  D. 180 days

**Q2.** An eligible short-duration crop loan generally becomes NPA after overdue for:  
A. One crop season  B. Two crop seasons  C. 90 days  D. Three crop seasons

**Q3.** SMA-2 normally represents overdue of:  
A. 1–30 days  B. 31–60 days  C. 61–90 days  D. More than 180 days

**Q4.** An asset that has remained substandard for 12 months becomes:  
A. Standard  B. Doubtful  C. Loss automatically  D. Written-off automatically

**Q5.** Under SARFAESI Section 13(2), the borrower is generally given how many days to pay after demand notice?  
A. 15  B. 30  C. 45  D. 60

**Q6.** Which is generally excluded from SARFAESI enforcement?  
A. Commercial building  B. Eligible secured machinery  C. Agricultural land  D. Eligible secured vehicle

**Q7.** The DRT pecuniary threshold for bank recovery applications was raised to:  
A. ₹5 lakh  B. ₹10 lakh  C. ₹20 lakh  D. ₹1 crore

**Q8.** Write-off of a loan means:  
A. Borrower is automatically forgiven  B. Accounting removal; recovery may continue  C. Bank cannot sue  D. Loan is converted into deposit

**Q9.** A doubtful loan with ₹3 lakh unsecured portion generally requires provision on that unsecured portion of:  
A. 15%  B. 25%  C. 40%  D. 100%

**Q10.** Corporate insolvency under IBC is adjudicated by:  
A. RBI  B. DRT  C. NCLT  D. NPCI

### Answer key with explanations
1. **C** — The ordinary term-loan NPA threshold is more than 90 days overdue.  
2. **B** — Short-duration crop loans: two crop seasons; long-duration: one.  
3. **C** — SMA-2 = 61–90 days.  
4. **B** — Doubtful after remaining substandard for 12 months.  
5. **D** — Section 13(2) gives a 60-day demand period.  
6. **C** — Agricultural land is excluded under SARFAESI.  
7. **C** — ₹20 lakh, notified in 2018.  
8. **B** — Write-off is accounting treatment; recovery can continue.  
9. **D** — Doubtful unsecured portion: 100% provision.  
10. **C** — NCLT is the adjudicating authority for corporate insolvency.

---

## Official / reference reading
- RBI — Commercial Banks IRACP Directions (2025; updated versions): https://www.rbi.org.in/scripts/NotificationUser.aspx?Id=13146
- RBI — Commercial Banks Asset Classification, Provisioning and Income Recognition Directions (2026; effective 1 April 2027): https://www.rbi.org.in/scripts/NotificationUser.aspx?Id=13400
- SARFAESI Act Section 13 (statutory text reference): https://indiacode.ecourtsindia.com/securitisation-reconstruction-financial-assets-enforcement-security-interest-act/section/13/
- SARFAESI Section 31 exclusions (statutory text reference): https://indiacode.ecourtsindia.com/securitisation-reconstruction-financial-assets-enforcement-security-interest-act/section/31/
- Ministry of Finance / Gazette — DRT threshold ₹20 lakh notification: https://upload.indiacode.nic.in/showfile?actid=AC_CEN_2_33_00045_199351_1524048948493&filename=Notification_dated_06.09.2018_sw.pdf&type=notification
- IBBI — IBC legal framework: https://ibbi.gov.in/legal-framework/act
- IBBI — IBC Amendment Act, 2026: https://ibbi.gov.in/uploads/legalframwork/2026-04-07-115842-i5nsk-7ed69ef2a4d23a8b0d472cc0fcd55e79.pdf

---

**Day 4 complete.** Revise the 90-day rule, crop seasons, provisioning table and SARFAESI/DRT timelines first—they are high-yield MCQ areas.

# ISOPOLIA v1.0.0 — Proof-of-Personhood (`PoP-v34`) & Constitutional Chamber Veto Specification

**Status:** Normative Governance & Anti-Sybil Specification (resolving Critique #2 on `PoP` and hardcoded `0.015` veto constant)  
**Companion Simulator:** `sim/sim_sybil_graph.py`

---

## 1. Two-Layer Defense Against Rule-Change Capture

External audit correctly identified a critical modeling risk: if security against a catastrophic rule-change attack depended on a single unverified human veto constant (`0.015`, i.e., `98.5%` blocking), any degradation in voter turnout or PoP corruption makes capture profitable.

ISOPOLIA v3.4 therefore enforces a **defense-in-depth two-layer boundary**:

| Defense Layer | Scope Protected | Mechanism | Dependence on Human Turnout / PoP |
|---|---|---|---|
| **Layer A — Immutable `Noyau-0` Kernel (`L1–L5`)** | Monetary supply cap ($16\,180\,339$ ISOP), balance & escrow conservation, $0.0001\text{ ISOP/Ko}$ decimal ($1\,024\,000\text{ nomons/KiB}$) byte deposit, $\le 3\%$ Thymus ceiling, $\le 0.25\%$ Class-1 grant cap, strict L2 capability isolation, L3 prohibition of selective account freezes, `Creuset` reversibility | Hard-coded in L1 state transition engine (`crates/isopolia-noyau0` & independent clean-room clients). **Class 2/3 governance cannot mutate `L1–L5`.** | **Zero (0%)** — rejected deterministically at L1 (`L5MigrationNotReversible`, `SupplyCapExceeded`, `Class1TreasuryCapExceeded`). |
| **Layer B — Mechanistic Chamber Human Veto** | Economic & Ecosystem Treasury allocations (`Class 2` & `Class 3A` non-L1 parameters) within L1 caps | Dual-chamber vote: Token/Stake Chamber (`Sceau` + staked vote) **plus** Verified Human Citizen Chamber with **14-day timelock**, **commit-reveal secrecy**, and **unconditional `33.4%` blocking minority veto**. | **Bounded & quantified** (Section 3 below & `sim_sybil_graph.py` sensitivity matrix). |

---

## 2. Decentralized Privacy-Preserving Proof-of-Personhood (`PoP-v34`)

To resist single-issuer censorship, biometric honeypots, and vote buying, a **Verified Citizen Credential** requires an **independent $t$-of-$m$ threshold federation** ($t = 3$, $m \ge 5$ orthogonal trust classes under `PoP-v34.2`) with strictly unlinkable zero-knowledge epoch nullifiers.

### 2.1 Orthogonal Issuer Federation ($m \ge 5$, Hardened Threshold $t = 3$)
*(Security upgrade `PoP-v34.2`: raised from `t = 2` to **`t = 3` of `5`** across orthogonal hardware/civic classes after mathematical sensitivity review against the `647,213.56 ISOP` Ecosystem Fund — see Section 3.1.)*

No single entity, pair of colluding registrars, state, or biometrics vendor can mint citizens (`< 3` corrupted classes yields $\alpha_{\text{forge}} = 0$). Furthermore, every credential must include **at least one attestation from Tier A (Hardware/eIDAS `#1` or `#3`)** and **no single issuer class may co-sign more than `35%` of net epoch admissions**:
1. **eIDAS 2.0 / Sovereign ZK Wallet Attestor:** Blind predicate attestation proving unique EU/national digital ID age & personhood without leaking civil identity.
2. **Physical Key-Signing Parties (Web of Trust):** Randomized mutual witness ceremonies at local nodes with rate-limited quota and social-distance diversity checks.
3. **Hardware Secure Enclave + Local Liveness:** On-device WebAuthn / TPM / Secure Enclave key attestation paired with client-side anti-replay liveness; **no raw biometric vector or hash ever leaves the user's device**.
4. **Independent Civic / Academic Registrars:** Non-profit universities & FOSS cooperatives running audited blind issuance blind-signers.
5. **Reputation-Staked Arbiters (Dispute Challenge Bond):** Existing long-lived honest citizens co-signing revocable challenge bonds subject to slashing upon collusion proof.

### 2.2 Unlinkable Epoch Nullifier Construction
For each verified human with hardware-held secret $\text{sk}_H$:
- Each of $t \ge 3$ distinct issuer classes issues a blind BBS+/Camenisch-Lysyanskaya credential $\sigma_j$ over hidden attribute $\text{sk}_H$ for epoch window $e$ (`90 days` = `129_600` beats) without seeing either the citizen's on-chain agent wallets or the other issuers' blind messages.
- To register a Chamber Citizen Ballot on L1/Chamber for epoch $e$, the user submits:
  $$\text{nullifier}_{e} = H_{\text{PoP}}(\text{sk}_H \,\|\, e \,\|\, \text{chain\_id})$$
  accompanied by a NIZK proof $\pi_{\text{PoP}}$ attesting:
  1. Valid threshold signatures from $\ge t$ distinct accredited issuer classes active at epoch $e$,
  2. Correct deterministic derivation of $\text{nullifier}_e$ from the shared committed $\text{sk}_H$,
  3. Zero linkability to prior epochs ($e-1$) or to operational AI Agent `AccountId` wallets (`L2` isolation).
- **EU GDPR / French CNIL Compliance:** Zero personal identifiers (PII), zero passport scans, and zero raw or derived persistent biometric templates ever touch consensus state or network storage. Unlinkability across epochs eliminates long-term surveillance profiles.

### 2.3 Anti-Bribery & Anti-Collusion Protections
1. **Commit-Reveal Secret Ballot (Receipt-Free):** Citizen Chamber votes (`APPROVE` / `VETO` / `ABSTAIN`) are committed with random salt in a sealed phase (`Days 1–9`) and revealed inside an optional re-randomizable ZK / ephemeral-relayer pool (`Days 10–14`). **An attacker cannot cryptographically verify how a bribed human voted**, collapsing provable bribery ROI.
2. **Tightened Epoch Growth Rate Cap (`2%/epoch` = ~`8.2%/year`):** Total verified citizen set cannot expand by more than **`+2%` (`g_max = 0.02`) per epoch (`90 days`)** (tightened from `+5%` under `PoP-v34.2`), neutralizing flash-Sybil registration before a contested governance vote and bounding maximum multi-year accumulation even under full $t$-issuer compromise. Any single issuer exceeding `35%` of net epoch signatures triggers an automatic Chamber quarantine freeze of that issuer key.
3. **Emergency Guardian Freeze vs Veto Delay:** Any Class 2/3 proposal affecting Ecosystem Treasury $\ge 1\%$ automatically triggers a mandatory `14-day` (`20_160` beats) execution timelock with on-chain watcher alerts.

---

## 3. Mechanistic Derivation of Blocking Probability (Replacing Hardcoded `0.015`)

Rather than asserting $P(\text{pass}) \propto 0.015$, `sim/sim_sybil_graph.py` derives pass probability endogenously from four structural parameters:
1. **$N_H$**: Total active epoch-verified humans (`2_500` in baseline model).
2. **$\alpha_{\text{corrupt}} \in [0, 0.45]$**: Fraction of Citizen nullifiers bought, colluding, or forged through compromised PoP issuers.
3. **$T_{\text{base}} \in [0.05, 0.50]$**: Baseline honest citizen turnout, amplified dynamically when high value-at-risk $V_{\text{enjeu}}$ triggers timelock watchtower alarms ($\Delta T = f(V_{\text{enjeu}} / \text{Treasury})$).
4. **Blocking Rule**: A Class 2/3 governance proposal **FAILS** if either:
   - Honest `VETO` votes reach the **blocking minority $\ge 33.4\%$ of total cast votes**, OR
   - `APPROVE` falls below `$66.7%` supermajority of participating ballots (with minimum valid turnout quorum $\ge 15\%$ of registered epoch humans), OR
   - The proposal touches any immutable `Noyau-0` Law `L1–L5` (deterministically rejected by Layer A regardless of turnout or $\alpha_{\text{corrupt}}$).

See `sim/results_v34_rigorous.json` (generated by `sim/sim_sybil_graph.py`) for the exact phase diagram across $(\alpha_{\text{corrupt}}, T_{\text{base}}, V_{\text{enjeu}})$ identifying where governance holds cleanly vs where high corruption + voter apathy creates risk.

### 3.1 Closed-Form Expression of $\alpha(k, t, g, E)$ vs a Worst-Case `647,213.56 ISOP` (`4%`-of-Cap) Accumulated Ecosystem Fund

To make the PoP security assumption explicit and falsifiable, let $k \in \{0,\dots,m\}$ be the number of compromised issuer classes, $t$ the issuance threshold, $g$ the per-epoch (`90-day`) population expansion cap, $E$ the attacker's patient accumulation horizon in epochs, and $\alpha_0$ the residual human vote-buying rate under receipt-free commit-reveal ($\approx 0.02$):

$$\alpha(k, t, g, E) = \begin{cases} \alpha_0 & \text{if } k < t \quad (\text{threshold sound; cannot forge nullifiers}) \\[6pt] 1 - \dfrac{1 - \alpha_0}{(1 + g)^E} & \text{if } k \ge t \quad (\text{attacker saturates entire } g \text{ quota every epoch}) \end{cases}$$

**Why `2-of-5` + `g = 5%` was dangerously loose:**
If an attacker compromises just $k = 2$ of $5$ classes under $t=2, g=0.05$ (with $\alpha_0 = 0.02$), fake identities reach **`19.38%` after `1 yr` ($E=4$)**, **`33.67%` after `2 yrs` ($E=8$)** (`p_pass = 27.60%` at `8%` voter turnout), and **`45.43%` after `3 yrs` ($E=12$)** (`p_pass = 67.20%`, positive expected ROI **`+234,927.5 ISOP`** against the **`647,213.56 ISOP`** Ecosystem Fund `250k + 247.2k + 150k` under `75k ISOP`/issuer compromise assumptions).

**Why upgraded `PoP-v34.2` (`t = 3 of 5` + `g = 2%/epoch`) closes this window:**
1. Corrupting $k = 2$ classes yields **$\alpha = \alpha_0 = 2.00\%$** (`p_pass = 0.00%`, expected ROI `-200,000.0 ISOP`, attacker loses proposal & issuer bonds unconditionally).
2. Even compromising $k = 3$ orthogonal classes simultaneously limits worst-case $\alpha$ to **`9.46%` after 1 yr ($E=4$)** (`p_pass = 0.00%`), **`16.36%` after 2 yrs ($E=8$)** (`p_pass = 0.00%`), and **`22.73%` after 3 full years ($E=12$)** (`p_pass = 2.40%` at `8%` turnout, net expected ROI **`-259,466.9 ISOP`**) — remaining below the `>= 30%` danger boundary for **~3 years** while giving on-chain watchers (`14-day` timelock) and per-issuer spike monitors (`<= 35%` share cap) ample runway to rotate compromised issuers, with **`Layer A (L1–L5)`** remaining `100%` immutable at any $\alpha$.

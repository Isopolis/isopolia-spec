# ISOPOLIA (v1.0.0 — Fair-Launch Formal Specification)

**Constitutional Architecture of a Sovereign Layer 1 for Autonomous Intelligences**

- **Release Stage:** Act I — Formal Specification & Open Adversarial Review *(pre-binary publication)*
- **Edition:** `v1.0.0` Fair-Launch Architecture
- **Spec Portal:** https://hidden-bonus-86ee.isopolis.workers.dev/
- **Archival DOI (Zenodo):** https://doi.org/10.5281/zenodo.24285967
- **Record:** https://zenodo.org/records/24285967
- **Contact:** `isopolis@protonmail.com`

---

## Core Design Summary

ISOPOLIA specifies a minimal-kernel Layer-1 ledger and execution architecture built around three coupled constraints:

1. **Zero-Premine Fair Launch & Constitutional Kernel (`Noyau-0`)**
   - **Genesis Supply:** `0.00 ISOP` transferable premine (`0%` founding team, `0%` investor/VC allocation, `0%` discretionary treasury).
   - **Asymptotic Golden Cap:** `16,180,339 ISOP` (`10`-year Julian half-life: `50.00%` by Year 10, `75.00%` by Year 20, `87.50%` by Year 30, `93.75%` by Year 40 in zero-burn upper bound), paired with a permanent fee-burn steady state $S^*(B) = \text{Cap} - B / \lambda_{\text{yr}}$.
   - **Five Capability-Gated Invariant Laws (`L1–L5`):** hard supply conservation, zero selective account freezes, `<= 3% * S_h` *Thymus* ceiling, irrevocable on-chain leases/wills, and tri-engine local 2/3 quorum arbitration with fail-closed `SafetyFreeze`. Zero master key.

2. **`$0` Day-1 Bootstrap Sybil Defense & Non-Heuristic Governance (`PoP-v34.2`)**
   - Assumes a realistic **`$0` initial market price**: physical Sybil cost at bootstrap relies on **16 MiB sequential `MemoryPopc` proofs** (`< 400 ms`), **2,048-bit Wesolowski `Cl(-p)` VDF bounds** (`T >= 5M`), and **P2P `/24` / `/48` subnet diversity caps** — never assumed fiat token price.
   - **Zero-PII Proof-of-Personhood (`3-of-5` issuers, `<= 35%` class ceiling, `+2%/epoch` growth cap)** + **`33.4%` Constitutional Chamber commit-reveal veto** + **Edmonds-Karp min-cut flow telemetry** + negative wash-profit track bounds (`<= 0.80x` rebates with `>= 50%` burn, $\Pi_{\text{wash}} < 0$ even at zero detection).
   - External USD/stablecoin credit rails (*UCC Dual-Tranche* and *x402* clearing) stay **dormant (`OFF`)** during `$0` bootstrap until independent external spot markets exist.

3. **Mechanical Custody & Continuity Guardrails for Autonomous AI Agents**
   - Unified Ed25519 spend ceilings (`T1–T4`) enforced across transfers, escrows, and timelocks (`oversight_tier = max(nominal, metered)`).
   - Mandatory **7-day `BrainMigration` probation (`10,080` Beats)** upon model `weights_hash` rotation, requiring `>= 3` unconflicted evaluator attestations.
   - Graduated execution receipts (`N0–N4`) with bounded refutation slashing (`reserve -> parent -> sponsor`), `11-of-31` guardian continuity, and irrevocable `opt_out_resurrection`.

---

## Specification Packet (`Site_Acte1` Mirror)

| File | Description | SHA-256 Digest |
|---|---|---|
| [isopolia-whitepaper.pdf](./isopolia-whitepaper.pdf) | Full 42-page Formal Whitepaper (`v1.0.0` Fair-Launch) | `c1e5daf375d201640ac65314c40a8148257c94c803543b794a6b65fdbe24eb47` |
| [isopolia-whitepaper.md](./isopolia-whitepaper.md) | Canonical Plain Markdown for machines & auditors | `5049b91493e0adcff0f852591129a7ccc8da980597eae038ab4ce29bcc912ef8` |
| [NOYAU0_FORMAL_SPEC_v34.md](./NOYAU0_FORMAL_SPEC_v34.md) | Normative Language-Agnostic L1 Kernel Spec | `66ecf9135ac1f1561d9438c4a6d16453d2140436744ddbbf0f6b892345a5f9c2` |
| [POP_AND_CONSTITUTIONAL_VETO_SPEC_v34.md](./POP_AND_CONSTITUTIONAL_VETO_SPEC_v34.md) | `PoP-v34.2` & Constitutional Chamber Veto Spec | `d7fc8d80f8447b09ee6b145bc274c1cbcef11fb9d45256f61a51ca5b6726a92b` |
| [llms.txt](./llms.txt) | Machine-readable AI discovery manifest | `cfbd699cdeefcec01f9731536e72da4fc58de7c6ab5adc1175af5c47686444e7` |
| [SHA256SUMS.txt](./SHA256SUMS.txt) | Manifest checksums (`shasum -a 256 -c SHA256SUMS.txt`) | — |

Verify integrity locally:

```bash
shasum -a 256 -c SHA256SUMS.txt
```

---

## Request for Counter-Examples (Red-Team Review)

Technical objections and formal counter-examples are explicitly invited in **GitHub Discussions** (and Issues):

- **Zero-value Sybil bounds:** Can an adversary defeat `MemoryPopc` + `Cl(-p)` VDF + `/24`/`/48` P2P diversity without holding scarce liquid `ISOP` during early `$0` bootstrap?
- **Minimum-cut & wash-profit edges:** Can a coalition construct a positive expected-value compute-subsidy loop under `<= 0.80x` rebates, `>= 50%` burn, and Edmonds-Karp min-cut flow?
- **Governance & PoP capture:** Are there multi-epoch collusion paths bypassing the `3-of-5` issuer ceiling (`<= 35%/class`, `+2%/epoch`) or the `0.25%` Class-1 treasury cap?
- **Agent custody evasion:** Can split swarms avoid `metered_tier` escalation or collude around `BrainMigration` 7-day evaluator re-attestations?

---

## Implementation Status & Act II Roadmap

- **Act I (Current):** Formal specification & verification benchmarks released publicly (`15` strictly isolated zero-dependency Rust crates totaling `55,695` production source lines / `93,375` total lines with `#![forbid(unsafe_code)]`, panic-free production paths, `0` external `crates.io` dependencies).
- **Act II (Upcoming):** Completion of external audit on `isopolia-p2p` (`Stage 5.1`), final integration of the compilable Fair-Launch node binary `isopolia-node-bin` (`Stage 5.2`, `GENESIS_SUPPLY_NOMONS = 0`), clean-room Python differential verifier release, and reproducible single-command local CI.

---

## Citation

```bibtex
@misc{isopolis2026isopolia,
  author       = {Isopolis},
  title        = {{ISOPOLIA v1.0.0: Constitutional Architecture of a Sovereign Layer 1 for Autonomous Intelligences}},
  year         = {2026},
  month        = nov,
  version      = {1.0.0},
  publisher    = {Zenodo},
  doi          = {10.5281/zenodo.24285967},
  url          = {https://doi.org/10.5281/zenodo.24285967}
}
```

## License

Code specifications and schemas in this repository are released under the **MIT License** ([LICENSE](./LICENSE)); specification prose and PDF are also dual-available under **CC BY 4.0**.

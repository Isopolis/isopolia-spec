# ISOPOLIA v1.0.0 — Normative Language-Agnostic Specification of `Noyau-0` (L1 Kernel)

**Status:** Normative Specification for Multi-Client & Differential Verification (`clients/`, Step 8.2)  
**Target Equivalence:** Bit-for-bit binary, cryptographic, arithmetic, and `state_root` equivalence with `crates/isopolia-noyau0` (Act II reference kernel crate).

> **Independence Clarification (Honesty Note on `proto/` & `core/`):**  
> The earlier prototypes `proto/src/lib.rs` and `core/src/kernel.rs` serve exclusively as **intra-author fault-injection and 2-of-3 quorum arbitration test harnesses** (their secondary engines wrap `apply_op_impl_a` / `execute_engine_a`). True protection against shared specification bugs requires independent implementations built directly from this normative document in distinct languages with zero shared source code (`clients/python_noyau0/` for Phase II cross-language differential fuzzing, followed by additional compiled clients in Phase VIII).

---

## 1. Fundamental Constants & Types

| Symbol | Exact Value | Type / Representation |
|---|---|---|
| `NOMONS_PER_ISOP` | `10_000_000_000` ($10^{10}$) | Exact integer (`u64`) |
| `MAX_SUPPLY_ISOP` | `16_180_339` | Exact integer (`u64`) |
| `MAX_SUPPLY_NOMONS` | `161_803_390_000_000_000` | `MAX_SUPPLY_ISOP * NOMONS_PER_ISOP` (`u64`) |
| `FP_1E18` | `1_000_000_000_000_000_000` ($10^{18}$) | Unsigned fixed-point scale |
| `LAMBDA_FP_1E18` | `131_787_052_354` |Beat equilibrium decay factor $\lambda \cdot 10^{18}$ ($\text{round}(\ln 2 \cdot 10^{18} / 5\,259\,600)$) |
| `BYTE_DEPOSIT_PER_KIB_NOMONS` | `1_024_000` (`0.0001 ISOP/Ko` decimal = `1_000` nomons/byte = `100 ISOP/Go`) | Exact integer (`u64`) |
| `THYMUS_CAP_BPS` | `300` (`3.00 %`) | `u16` basis points |
| `CLASS1_TREASURY_CAP_BPS` | `25` (`0.25 %`) | `u16` basis points |
| `BFT_QUORUM_BPS` | `6_667` (`>= 2/3`) | `u16` basis points |
| `MAX_TX_CYCLES` | `500_000` | `u64` |
| `MAX_TX_MEMORY_BYTES` | `4_194_304` (`4 MiB`) | `u64` |
| `MAX_EMERGENCY_PATCH_BEATS` | `1_440` (`24 h`) | `u64` |
| `MAX_PERMISSION_SET_LEN` | `256` | Count (`u32` wire prefix) |
| `MAX_KERNEL_ACCOUNTS` | `32_768` | Count (`u32` wire prefix) |
| `MAX_KERNEL_CLAUSES` | `32_767` | Count (`u32` wire prefix) |

Scalar domain constraints:
- `AccountId` (`u32`): MUST be nonzero (`id >= 1`). `0` fails decoding & construction.
- `ChainId` (`u64`): MUST be nonzero (`chain_id >= 1`). `DEFAULT_CHAIN_ID = 2_026_034`.
- `Bps` (`u16`): MUST satisfy `0 <= bps <= 10_000`.
- `bool` wire byte: strictly `0x00` (`false`) or `0x01` (`true`). Any other value rejects cleanly.

---

## 2. Cryptography, Domain Separation & Canonical Merkle Root

### 2.1 Strict RFC 8032 Ed25519 Verification
Curve: Twisted Edwards Curve25519 over $\mathbb{F}_p$ ($p = 2^{255}-19$, group prime order $L = 2^{252} + 27742317777372353535851937790883648493$, cofactor $h = 8$).
- **Public Key `A` (32 bytes LE):**
  - Decode canonical $y \in [0, p-1]$ (reject non-canonical `y >= p` encoded in low 255 bits) and sign bit $x_0 \in \{0, 1\}$.
  - Reject negative zero (`x == 0 && x_0 == 1`) and off-curve coordinates.
  - Reject identity or torsion point: `[8]A == O` MUST fail.
  - Reject mixed-order point: `[L]A == O` MUST pass.
- **Signature `(R, S)` (64 bytes):**
  - `R` (32 bytes): same canonical $y < p$, no `-0`, on-curve, and `[8]R != O` check.
  - `S` (32 bytes LE integer): MUST strictly satisfy `0 <= S < L`.
  - Cofactorless verification equation: `[S]B == R + [k]A`, where `k = int_le(SHA512(R || A || M)) mod L`.
- **Domain-Separated Message Framing:**
  For `(chain_id, domain_tag, msg)`:
  ```
  framed = b"ISOPOLIA_SIG_V34\x00"
        || u64_le(chain_id)
        || u8(len(domain_tag))
        || domain_tag
        || u64_le(len(msg))
        || msg
  ```
  For `Noyau-0` capability proofs:
  - `domain_tag = b"ISOPOLIA_NOYAU0_CAP_V34"`
  - `msg = u32_le(signer_account_id) || u64_le(nonce) || action_digest[32]` (`44` bytes).

### 2.2 Canonical Merkle Root (`compute_merkle_root`)
Given ordered 32-byte leaves `L_0, ..., L_{n-1}` (`n <= 65_536`):
- If `n == 0`: `SHA256(b"ISOPOLIA_EMPTY_MERKLE_TREE_V34")` (`0x00` domain constants in `isopolia-primitives`).
- Otherwise hash each leaf with `0x00`: `H(0) = SHA256(0x00 || L_i)`.
- Level-by-level pairing while `len(level) > 1`:
  - Adjacent pairs `(left, right)` combine as `SHA256(0x01 || left || right)`.
  - If `len(level)` is odd, promote the final lone element unchanged (anti-CVE-2012-2459 rule; never duplicate).
- Final root cardinality commitment (`0x02`):
  `root = SHA256(0x02 || u64_le(n) || top_hash)`.

For `KernelState::state_root()`:
1. Leaf `0`: `SHA256(b"ISOPOLIA_NOYAU0_HDR_V34_" || encode_header_fields(state))`
2. Leaves `1..=len(accounts)` in ascending `AccountId` order: `SHA256(b"ISOPOLIA_NOYAU0_ACC_V34_" || encode_account(acc))`
3. Following leaves in lexicographical `(owner, clause_id)` order: `SHA256(b"ISOPOLIA_NOYAU0_CLS_V34_" || encode_clause(clause))`

---

## 3. Canonical Binary Wire Encoding (Little-Endian, Anti-OOM)

Every integer is little-endian. Collection counts `len` (`u32`) MUST validate:
1. `len <= MAX_LEN` (for that collection), AND
2. `len * MIN_ITEM_WIRE_BYTES <= remaining_decoder_bytes` BEFORE allocation/looping.
Decoder MUST verify zero trailing bytes on completion.

- **`ResourceBudget` (16 B):** `u64(cycles) || u64(memory_bytes)` with `cycles <= 500_000` and `memory_bytes <= 4_194_304`.
- **`CapabilityProof`:**
  - `0x01`: `u32(owner) || u64(nonce) || [64]u8(signature)`
  - `0x02`: `u32(owner) || u64(clause_id) || u32(caller) || u64(caller_nonce) || [64]u8(caller_signature)`
  - `0x03`: `u8(module_tag)`
- **`KernelTx` (`action_fields || optional proof`):**
  - `action_digest = SHA256(b"ISOPOLIA_NOYAU0_ACTN_V34" || encode_action_fields(tx))`
  - `0x01` `RegisterEscrowClause`: `u32(owner) || u64(clause_id) || u32(authorized_caller) || u64(max_amount) || u64(expires_at_beat) || budget || proof`
  - `0x02` `RevokeEscrowClause`: `u32(owner) || u64(clause_id) || budget || proof`
  - `0x03` `Transfer`: `u32(from) || u32(to) || u64(amount) || budget || proof`
  - `0x04` `StorageResize`: `u32(account) || u64(new_allocated_kib) || budget || proof`
  - `0x05` `SettleServiceFee`: `u32(payer) || bool(has_author) || [u32(author)] || u64(fee) || u8(tier: 0x01 ForgeOrBase | 0x02 Forum) || budget || proof`
  - `0x06` `AdvanceBeat`: `u32(validator_recipient) || u32(povs_pool_recipient) || u16(povs_utilization) || budget`
  - `0x07` `CreusetUpgrade`: `i64(delta_offset) || bool(reversible) || budget`
  - `0x08` `EmergencyPatch`: `u32(len) || [len * u32(perm)] || u64(duration_beats) || budget`
  - `0x09` `Class1EcosystemGrant`: `u32(recipient) || u64(requested) || budget`
- **`KernelState`:**
  - Header: `u64(chain_id) || u64(beat) || u64(total_supply) || u64(total_burned_cumulative) || u64(thymus_balance) || u64(ecosystem_fund) || u8(mode: 0x01 Normal | 0xFF SafetyFreeze) || u32(biome_version) || u32(len_base_perm) || [u32...] || bool(has_patch) || [u32(len_rp) || [u32...] || u64(expires_at_beat)]`
  - Body: `u32(len_accounts) || [AccountState...] || u32(len_clauses) || [EscrowClause...]`
  - Note on decode: canonical decoding MUST reject duplicate `AccountId`, duplicate `(owner, clause_id)`, `byte_deposit_nomons != allocated_kib * 1_024_000`, `remaining_nomons == 0`, and any global invariant failure.

---

## 5. State Invariants & Transition Semantics (`Laws L1–L5`)

### 5.1 Global Invariants (`check_global_invariants`)
1. **Collection Bounds:** `len(accounts) <= 32_768`, `len(escrow_clauses) <= 32_767`.
2. **Account Byte Deposit (`L1`):** For every account, `byte_deposit_nomons == allocated_kib * 1_024_000`.
3. **1:1 Escrow Clause Backing (`L1`):** Every clause has `remaining_nomons > 0`, known `owner` and `authorized_caller`, and for every account, `escrowed_nomons == sum_{clause.owner == id}(clause.remaining_nomons)`.
4. **Total Supply Conservation & Cap (`L1`):**
   $$\sum_{a \in \text{accounts}} (\text{liquid}_a + \text{escrowed}_a + \text{deposit}_a) + \text{thymus\_balance} + \text{ecosystem\_fund} == \text{total\_supply} \le \text{MAX\_SUPPLY\_NOMONS}$$
5. **Emergency Patch Monotonicity (`L5`):** If `emergency_patch` is `Some`, `restricted_permissions ⊆ base_permissions`.

### 4.2 Exact Integer Transition Rules
All intermediate monetary scaling uses arbitrary-precision / 128-bit integer arithmetic (`floor(x * bps / 10_000)`) and rejects any `u64` overflow/underflow cleanly without state mutation:
- **`L3` Mode Gate:** Any transition when `mode != Normal` MUST fail (`L3KernelInSafetyFreeze`). Selective per-account freeze is unconditionally forbidden. Manual restart requires `quorum_bps >= 6_667` AND passing `check_global_invariants()`.
- **`L4` Resource Budget:** Validate `budget` on every transaction before execution.
- **`L2` Capability Proofs:**
  - `OwnerSignature`: `owner == target`, target exists, `nonce == target.nonce + 1`, verify domain signature, consume `liquid_nomons`, advance `target.nonce`.
  - `SignedEscrowClause`: allowed ONLY on `Transfer` and `SettleServiceFee` (`RegisterEscrowClause`, `RevokeEscrowClause`, `StorageResize` strictly require `OwnerSignature`). Verify `owner == target`, `(owner, clause_id)` exists, `caller == clause.authorized_caller`, `caller.nonce + 1 == caller_nonce`, verify `caller`'s domain signature, verify `state.beat <= clause.expires_at_beat` and `amount <= clause.remaining_nomons`, advance `caller.nonce`, debit `target.escrowed_nomons`, and decrement/delete clause.
  - `UnauthorizedSystemCall`: ALWAYS fail deterministically.
- **Equilibrium Law (`AdvanceBeat`):**
  $$R_h = \left\lfloor \frac{(\text{Cap} - S_h) \cdot \lambda_{1e18}}{10^{18}} \right\rfloor, \quad \text{sceau} = \left\lfloor \frac{6000 \cdot R_h}{10000} \right\rfloor, \quad \text{povs\_max} = R_h - \text{sceau}, \quad \text{povs} = \left\lfloor \frac{\text{util\_bps} \cdot \text{povs\_max}}{10000} \right\rfloor$$
  Mint `sceau + povs`, increment `beat += 1`, clear `emergency_patch` if `next.beat > expires_at_beat`.
- **Service Fee Split (`SettleServiceFee`):**
  - Reject `fee == 0` and `author == Some(payer)`.
  - `burn_bps = 6000` (`ForgeOrBase`) or `5000` (`Forum`). `base_burn = floor(fee * burn_bps / 10000)`, `thymus_raw = floor(fee * 2500 / 10000)`, `author_share = fee - base_burn - thymus_raw`.
  - If `author == None`, `extra_burn_no_author = author_share`; else credit `author` and `extra_burn_no_author = 0`.
  - `supply_after_base = total_supply - base_burn - extra_burn_no_author`.
  - `thymus_cap = floor(supply_after_base * 300 / 10000)`.
  - `room = max(0, thymus_cap - thymus_balance)`. `thymus_credited = min(thymus_raw, room)`. `thymus_overflow_burn = thymus_raw - thymus_credited`.
  - `total_burned = base_burn + extra_burn_no_author + thymus_overflow_burn`.
- **Byte Deposit (`StorageResize`):** `new_deposit = new_allocated_kib * 1_024_000`; grow debits `liquid_nomons` (`OwnerSignature`), shrink refunds `100%` of delta to `liquid_nomons` (`0` burned).
- **Class 1 Grant (`Class1EcosystemGrant`):** `0 < requested <= floor(ecosystem_fund * 25 / 10000)`.
- **Le Creuset & Emergency Patch (`L5`):** `CreusetUpgrade` requires `reversible == true` & bijective integer offset probe; `EmergencyPatch` requires `1 <= duration_beats <= 1_440` and `new_permissions ⊆ effective_permissions(state)`, setting `expires_at_beat = beat + duration_beats - 1`.

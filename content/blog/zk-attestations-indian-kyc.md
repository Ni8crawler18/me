---
title: "ZK Attestations vs Traditional KYC: A Practical Guide"
date: 2025-02-18
draft: false
summary: "Why most KYC flows only need a boolean answer, and how ZK-SNARKs, STARKs, and Groth16 can replace raw data sharing — with implementation tradeoffs."
tags:
  - "ZK Proofs"
  - "Privacy"
  - "Cryptography"
---

Most KYC flows consuming raw Aadhaar data only need a boolean answer: *Is this person over 18? Is their document valid? Do they reside in state X?*

Yet the standard integration pulls the full XML — name, address, photo, biometrics — into the relying party's database. This is unnecessary data exposure, and it creates a liability surface that scales with every integration partner.

This post covers the problem, the ZK-based alternative, the major proving systems you can use today, and the tradeoffs between them.

## The Problem with Traditional Attestations

Traditional identity verification follows a simple but flawed model:

1. User shares raw personal data (Aadhaar XML, PAN, bank statements)
2. Relying party extracts the fields they need
3. Relying party stores everything — including fields they never needed

This creates three problems:

- **Over-collection**: A lending app checking age eligibility now holds your full address, photo, and document number
- **Liability multiplication**: Every integration partner becomes an attack surface. India saw 1.4 billion records leaked in 2023 alone
- **Compliance burden**: Under DPDP Act 2023, every byte of personal data stored requires lawful purpose, consent management, and deletion infrastructure

The fix is conceptually simple: if you only need a yes/no answer, you should only receive a yes/no answer — with cryptographic proof that it's correct.

## Zero-Knowledge Proofs: The Core Idea

A zero-knowledge proof lets a prover convince a verifier that a statement is true without revealing *why* it's true. In our context:

> "I can prove I am over 18 without showing you my date of birth."

The proof is:
- **Complete**: If the statement is true, an honest verifier will be convinced
- **Sound**: If the statement is false, no cheating prover can convince the verifier (except with negligible probability)
- **Zero-knowledge**: The verifier learns nothing beyond the truth of the statement

For KYC, this means replacing data-heavy workflows with **boolean attestations** — single-bit answers backed by cryptographic guarantees.

## Proving Systems: The Landscape

Three families dominate production ZK systems today. Each makes different tradeoffs on proof size, verification time, prover cost, and trust assumptions.

### Groth16 (zk-SNARKs)

Groth16 is the most deployed SNARK system. It produces the smallest proofs and the fastest verification — but requires a trusted setup ceremony per circuit.

**How it works**: The prover computes a witness (private inputs satisfying the circuit), then generates an elliptic curve proof using a structured reference string (SRS) from the trusted setup. The verifier checks a pairing equation.

**Characteristics**:
- Proof size: ~128 bytes (3 group elements)
- Verification time: ~10ms (constant, regardless of circuit size)
- Prover time: O(n log n) where n = circuit constraints
- Trust assumption: Requires per-circuit trusted setup (toxic waste must be destroyed)

**Libraries**:
- [Circom](https://docs.circom.io/) + [snarkjs](https://github.com/iden3/snarkjs) — most mature toolchain for Groth16
- [Arkworks](https://github.com/arkworks-rs) (Rust) — flexible, research-grade
- [gnark](https://github.com/Consensys/gnark) (Go) — production-focused, fast prover

**Best for**: On-chain verification (Ethereum), mobile/browser proving where proof size and verification cost matter. This is what we used for Xylem — in-browser WASM proving in ~500ms.

### PLONK (Universal SNARKs)

PLONK introduced a universal trusted setup — one ceremony works for any circuit up to a given size. This eliminated Groth16's biggest operational headache.

**How it works**: Uses a polynomial commitment scheme (KZG) with a single universal SRS. The circuit is compiled into polynomial constraints, and the proof shows these polynomials satisfy the required identities.

**Characteristics**:
- Proof size: ~400-600 bytes
- Verification time: ~15-20ms
- Prover time: Slightly slower than Groth16
- Trust assumption: Universal trusted setup (one ceremony for all circuits)

**Libraries**:
- [Halo2](https://github.com/zcash/halo2) (Rust) — Zcash's PLONK variant, no trusted setup with IPA
- [plonky2](https://github.com/0xPolygonZero/plonky2) (Rust) — Polygon's recursive PLONK, very fast prover
- [gnark](https://github.com/Consensys/gnark) also supports PLONK

**Best for**: Systems where you update circuits frequently (no new ceremony needed), or where you want a single setup for multiple applications.

### STARKs (Transparent Proofs)

STARKs require **no trusted setup at all**. They rely on hash functions instead of elliptic curves, making them post-quantum secure. The tradeoff is larger proofs.

**How it works**: The computation trace is encoded as a polynomial, then the prover commits to it using a Merkle tree (FRI protocol). The verifier checks random evaluations. Security comes from collision-resistant hashing, not algebraic assumptions.

**Characteristics**:
- Proof size: ~40-200 KB (orders of magnitude larger than SNARKs)
- Verification time: ~50-100ms (logarithmic in circuit size)
- Prover time: Fast, highly parallelizable
- Trust assumption: **None** (transparent setup)

**Libraries**:
- [Winterfell](https://github.com/facebook/winterfell) (Rust) — Meta's STARK library
- [Stone](https://github.com/starkware-libs/stone-prover) — StarkWare's production prover
- [Miden](https://github.com/0xPolygonMiden/miden-vm) — Polygon's STARK VM

**Best for**: Applications where trust minimization is paramount, where proof size doesn't matter (off-chain verification), or where post-quantum security is required.

## Comparison Table

{{< sketch "proving-systems-tree" "Three production-ready families. The right pick depends on which constraint hurts most — proof size, setup ceremony, or post-quantum threat." >}}

| Property | Groth16 | PLONK | STARKs |
|---|---|---|---|
| Proof size | ~128 B | ~500 B | ~50-200 KB |
| Verification | ~10ms | ~20ms | ~100ms |
| Trusted setup | Per-circuit | Universal | None |
| Post-quantum | No | No | Yes |
| Prover cost | Medium | Medium | High (but parallelizable) |
| Maturity | High | High | Medium |

## Implementation: ZK KYC with Circom + Groth16

Here's how we implemented it for Xylem — the Aadhaar XML never leaves the browser, only the proof does:

{{< sketch "xylem-flow" "Two inputs converge into a Circom circuit, which compiles down to a Groth16 proof generated entirely client-side. The relying party only ever sees the proof." >}}

### The Circuit

A simplified age-check circuit in Circom:

```
template AgeCheck() {
    signal input birthYear;
    signal input birthMonth;
    signal input birthDay;
    signal input currentYear;
    signal input currentMonth;
    signal input currentDay;
    signal input minAge;
    signal output eligible;

    // Compute age in a ZK-friendly way
    signal yearDiff;
    yearDiff <== currentYear - birthYear;

    // Birthday already passed this year?
    signal monthCheck;
    monthCheck <== currentMonth - birthMonth;

    // Final eligibility: yearDiff >= minAge
    // (simplified — production circuit handles edge cases)
    signal ageOk;
    ageOk <== yearDiff - minAge;

    eligible <== 1;
}
```

The real circuit also verifies the UIDAI digital signature on the XML, ensuring the birth date hasn't been tampered with. The private inputs (birth date, XML data) stay in the browser. Only the proof and the boolean result leave.

### Performance

On a mid-range laptop (M1 MacBook Air):
- Circuit compilation: ~2s (one-time)
- Witness generation: ~100ms
- Proof generation (WASM): ~500ms
- Proof verification (server): ~10-50ms
- Proof size: < 1KB

These numbers make it viable for real-time KYC flows. The user waits less than a second.

## When to Use What

**Use Groth16 when**:
- You need the smallest possible proof (on-chain, bandwidth-constrained)
- Your circuit is stable and won't change often
- You can run a trusted setup ceremony (or use an existing one)

**Use PLONK when**:
- You iterate on circuits frequently
- You want one setup ceremony for multiple applications
- Slightly larger proofs are acceptable

**Use STARKs when**:
- You cannot trust any setup ceremony
- Post-quantum security is a requirement
- Proofs are verified off-chain (proof size doesn't matter)
- You need fast, parallelizable proving for large computations

## Use Cases Beyond KYC

The same boolean attestation pattern applies broadly:

- **Credit scoring**: "This person's credit score is above 700" without revealing the exact score or underlying financial data
- **Age verification**: Prove age eligibility for regulated services without sharing birth date
- **Employment verification**: "This person works at company X" without revealing salary, role, or tenure
- **Regulatory compliance**: Prove DPDP Act compliance without exposing the data being protected
- **Supply chain**: Prove a component meets specifications without revealing proprietary manufacturing parameters

## DPDP Act Alignment

India's Digital Personal Data Protection Act 2023 mandates:

1. **Purpose limitation**: Data collected must be limited to what's necessary
2. **Data minimization**: Process only what's adequate and relevant
3. **Storage limitation**: Delete data when the purpose is fulfilled

ZK attestations satisfy all three by construction — no personal data is collected, processed, or stored by the verifier. The proof itself contains zero information about the underlying data.

This isn't just a technical improvement. It's a fundamentally different compliance posture: instead of building deletion pipelines and consent management for data you shouldn't have collected in the first place, you never collect it.

## What's Next

We're working on extending this to multi-predicate attestations — proving several properties about a user in a single proof (age AND residency AND income bracket) without revealing any of the underlying values. The circuit complexity grows, but Groth16 proof size stays constant.

The tooling is maturing fast. Two years ago, writing Circom circuits required deep cryptographic knowledge. Today, libraries like `circomlib` provide tested building blocks for common operations — comparators, hash functions, signature verification — that you compose like Lego.

If you're building anything that touches personal data verification, ZK attestations should be your default assumption. The question isn't whether the technology is ready — it is. The question is how quickly existing workflows can be refactored to stop collecting data they never needed.

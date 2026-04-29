---
title: "PII Exposure Isn't a Storage Problem — It's a Collection Problem"
date: 2026-04-29
draft: false
summary: "Encryption-at-rest, access controls, and breach-response playbooks treat PII exposure as a storage problem. The real fix is structural: stop collecting what you can prove without."
tags:
  - "Privacy"
  - "ZK Proofs"
  - "Cryptography"
  - "DPDP"
---

The standard response to a PII breach is more controls. Encrypt the database. Rotate the keys. Tighten the IAM policies. Buy a SIEM. Train the staff. None of this addresses the underlying issue: the data was collected in the first place, and once collected, every control becomes a probabilistic bet.

This post argues that PII exposure is a *collection* problem, not a *storage* problem — and that zero-knowledge proofs are the only widely-available primitive that lets you stop collecting without breaking the verification flows that depend on the data.

## The Lifecycle of Collected PII

Once a system collects a piece of PII, it enters a lifecycle that almost always ends in exposure. The interesting question is not *if* but *when* and *how much*.

{{< sketch "pii-lifecycle" "Every stage between collection and leakage adds attack surface — and most of them are governance problems no encryption fixes." >}}

A typical Indian fintech onboarding flow collects:

- Aadhaar XML (name, DOB, address, photo, gender, mobile hash)
- PAN (name, DOB, father's name)
- Bank statement (account number, transactions, balance, counterparties)
- Selfie + liveness video
- Device fingerprint, IP, geolocation

This data is needed for *one decision* — should we onboard this user? — and then it lives in the system for the next 7 years (RBI retention) or longer. During that time it is:

1. **Replicated** to backups, analytics warehouses, fraud-scoring vendors, regulator submissions
2. **Re-purposed** for marketing segments, lookalike audiences, cross-product underwriting
3. **Accessed** by employees, contractors, customer support, ML pipelines, BI dashboards
4. **Aggregated** with other datasets via probabilistic joins (name + DOB + city is a near-unique key)
5. **Eventually leaked** — through misconfigured S3 buckets, stolen laptops, insider exfiltration, vendor compromise, or regulator subpoena

Encryption protects against exactly one of these — the stolen-laptop / cold-storage-leak case. The other four are governance problems that no amount of cryptography can fix once the data exists.

## Why Data Minimization in Practice Fails

DPDP Act 2023, GDPR, and CCPA all require data minimization. None of them are particularly effective at it, for three structural reasons:

**1. The verifier needs to recompute the predicate.** A KYC reviewer can't decide if a user is over 18 unless they see the date of birth. Once they've seen it, the data is in the system. Minimization frameworks assume the verifier can be trusted to "use it and forget it" — humans and databases don't work that way.

**2. Audit trails require retention.** Regulators want to see *what* you verified, not just *that* you verified. To prove you checked age eligibility, you historically had to store the DOB you checked against. The compliance evidence and the PII are the same record.

**3. ML pipelines metabolize PII.** Once a piece of PII enters the analytics layer, it is featurized, embedded, joined, and partially reconstructed across derived tables. Deleting the source row doesn't delete the model trained on it.

Each of these can be patched with policy. None of them can be eliminated with policy. The structural fix is to never collect the data, and that is exactly what zero-knowledge proofs make possible.

## The ZKP Reframe: Predicates, Not Records

A zero-knowledge proof attests to a *predicate* over data without revealing the data. The relying party gets a single bit (or a small fixed-size assertion) plus a cryptographic proof that the bit is correct.

Instead of:

```
GET /aadhaar?id=XXXX
→ { name, dob, address, photo, gender, mobile_hash }
→ verifier checks dob, stores everything
```

You get:

```
POST /verify_predicate { predicate: "age >= 18", proof: 0x... }
→ verifier runs verify(proof) → true
→ verifier stores: { predicate, proof_hash, timestamp }
```

The compliance record is *the proof*, not the underlying data. The proof is non-malleable, timestamped, and contains zero information about the inputs. You have audit-grade evidence and zero PII.

{{< sketch "traditional-vs-zkp" "Same Aadhaar input, two architectures: one stores everything and waits to be breached; the other stores a proof and has nothing to lose." >}}

This is not a quantitative improvement over encryption — it is a qualitatively different threat model. The breach scenario becomes uninteresting because there is nothing of value to leak.

## The Predicate Patterns That Cover 80% of PII Use Cases

Most production PII use cases reduce to one of four ZK predicate patterns. Each has well-understood circuits and library support today.

{{< sketch "four-predicate-patterns" "Range, set membership, hash preimage, and aggregations — composable building blocks for nearly every PII verification need." >}}

### 1. Range Proofs — "Value is in interval [a, b]"

The single most common PII use case. Examples:

- Age ≥ 18, ≥ 21, ≥ 65
- Income between ₹X and ₹Y for tax bracket / loan eligibility
- Credit score above threshold
- Account balance above minimum for premium service tier
- Distance from a location below threshold (geofencing)

Bulletproofs are optimized specifically for this — proof size scales logarithmically with the bit-width of the value. For 64-bit integers, proofs are around 700 bytes. Groth16 also handles range proofs efficiently if you bake the comparator into the circuit.

### 2. Set Membership — "Value is in set S"

Proves a value belongs to a known set without revealing which element.

- "This Aadhaar is in the UIDAI database" (no need to send the Aadhaar)
- "This document hash is on the revocation list" (or *not* on it)
- "This user is in the allowlist of beta testers"
- "This wallet address is sanctioned" (or not)

Implemented with Merkle proofs inside a SNARK. The set is committed to a Merkle root; the proof shows a valid path from the user's leaf to the root. Set size has no effect on proof size — only on prover time.

### 3. Hash Preimage / Signature Verification — "I know x such that H(x) = y"

The building block underneath most identity attestations.

- "I know the Aadhaar XML signed by UIDAI whose name matches this commitment"
- "I know the password whose bcrypt hash is stored on this server" (passwordless auth)
- "I know the private key for this public key" (without signing a challenge that could be replayed)

Poseidon and MiMC are the SNARK-friendly hash functions of choice. SHA-256 inside a SNARK is expensive (~30k constraints) but doable when you must interop with non-SNARK systems.

### 4. Aggregations — "Sum / count / average over a set satisfies P"

Proves statistical properties of a private dataset.

- "My average monthly balance over the last 6 months is ≥ ₹50,000" (loan underwriting without sharing transactions)
- "I have made at least 12 transactions with this counterparty" (relationship strength without transaction history)
- "My carbon footprint is below threshold X" (ESG attestation without supply chain disclosure)

These compose the previous three with arithmetic constraints. Circuit complexity grows linearly with the dataset size, which makes them practical for hundreds-to-thousands of records, not millions.

## A Worked Example: Income Bracket Without Income Disclosure

Consider a government subsidy that requires household income below ₹3,00,000/year. The traditional flow:

1. User uploads ITR / bank statements
2. Reviewer reads the document, confirms income, approves
3. Document sits in the system for 7 years; reviewer remembers the income; analytics pipeline learns the income distribution

The ZK flow:

1. User's bank issues a signed attestation: `(account_id, annual_income, signature)`
2. User runs a circuit locally that:
   - Verifies the bank's signature on the attestation
   - Asserts `annual_income < 300_000`
   - Outputs `eligible = 1`, `attestation_commitment = H(account_id || nonce)`
3. User submits `(eligible, commitment, proof)` to the subsidy portal
4. Portal verifies the proof, stores the commitment as the audit record

What the portal knows: the user is eligible, and a commitment they can verify against in case of dispute. What the portal does not know: the income, the bank account, the actual amount.

The same circuit works for any income threshold. The same pattern works for credit scores, asset values, employment tenure, transaction counts.

## Operational Reality: What Breaks

ZKP isn't free. Three categories of friction matter for production systems.

**1. Issuer cooperation.** The bank, the government, the employer — whoever is the source of truth — has to issue data in a SNARK-friendly format (signed attestations the user controls, ideally with selective disclosure). Without this, you're back to scanning PDFs and re-introducing the disclosure problem at the parsing layer. India is unusual in that UIDAI-signed Aadhaar XML already supports this pattern; most other identity issuers worldwide do not.

**2. Revocation.** A proof generated last month doesn't know that the underlying credential was revoked yesterday. You either need short-lived attestations (issuer must be online), revocation lists (set non-membership proof), or accumulator schemes (more complex, smaller proofs). None of these are solved at the protocol level — every deployment makes its own tradeoff.

**3. Disputes and forensics.** When something goes wrong — fraud, regulatory inquiry, account dispute — the system needs to be able to reconstruct what happened. With raw PII, you read the database. With ZK attestations, you need either (a) the user to cooperate by re-revealing under a different proof, (b) escrow keys that defeat the privacy guarantee, or (c) accept that some forensics paths are simply closed. The right answer depends on regulatory regime and is the single most contentious design decision in real deployments.

## How This Maps to DPDP Act

DPDP Act 2023's data principal rights — right to access, correction, erasure, grievance redressal — assume the data fiduciary holds personal data. ZK attestation–based systems hold *no personal data*, which collapses several rights into trivial cases:

- **Erasure**: Already done. There is nothing to erase.
- **Access**: The data principal can regenerate the same proof from their own credentials at any time.
- **Correction**: Issued at the source (the bank, UIDAI), not at the verifier.
- **Purpose limitation**: The proof is bound to a specific predicate. It cannot be repurposed because there is no underlying data to repurpose.

The compliance burden inverts. Instead of building deletion pipelines, consent dashboards, breach notification workflows, and DPO offices for data you accumulated, you build a smaller surface that proves you never accumulated it.

## Where the Industry Is Today

The cryptographic primitives are production-ready. The tooling is not yet developer-friendly enough for general adoption — writing Circom or Halo2 circuits still requires understanding constraint systems, witness generation, and trusted setup ceremonies.

Three trends are closing this gap fast:

1. **Domain-specific circuit libraries** (zk-email for DKIM, zk-passport for ICAO MRTD, Anon Aadhaar for UIDAI XML) reduce common attestations to library calls
2. **zkVMs** (Risc0, SP1, Jolt) let you write proofs in Rust or Python, paying a constant-factor overhead in exchange for general-purpose programmability
3. **Hardware acceleration** (Ingonyama, Cysic, Fabric) is bringing prover times down by 10-100×, removing the last UX barrier for client-side proving

The window where "we encrypt our PII at rest" is an acceptable answer is closing. The next-generation answer — *we don't have your PII* — is now technically possible and increasingly affordable.

## What to Build Next

If you are designing a new system that touches PII today, the question is no longer whether to use ZKP — it is which predicates can be reduced to proofs, what your issuer ecosystem looks like, and where the unavoidable disclosure boundaries are.

The unavoidable boundaries are smaller than they look. Most systems collect PII out of architectural inertia, not necessity. The hard part is auditing each field against the actual decision it informs and asking: *would a single-bit attestation be enough?*

For most fields, the answer is yes. The infrastructure to deliver that bit, with cryptographic backing and zero exposure, is now mature enough to deploy. The remaining work is design discipline.

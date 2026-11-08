# A Layered Decentralized Architecture for Election Integrity

## Exact Tallies, Tamper-Evident Custody, and a Hyperdimensional Shadow Check

**Mark Krzan**
Version 2.0 — November 2026

---

## 1. Executive Summary

Public trust in electoral systems rests on verifiable integrity. Election infrastructure faces a genuine systems engineering paradox: the legal requirement for absolute voter anonymity conflicts with the engineering requirement for an unbroken, end-to-end chain of custody. Current systems bridge this gap with proprietary, closed-source hardware and administrative procedure — paper trails and protocols whose correctness the public is asked to take on faith.

An earlier version of this paper proposed replacing that architecture wholesale with Hyperdimensional Computing (HDC) over a decentralized mesh. Independent review found the proposal's core mechanism unsound for its stated purpose: a tally recovered from bundled hypervectors by similarity readout is *approximate*, and elections are one of the few engineering domains where approximate is a failing grade. This revision keeps the original goal — open, decentralized, edge-based election infrastructure with no closed-source vendor in the loop — and fixes the architecture the way systems engineering fixes anything: by separating concerns. No single mathematical mechanism is asked to carry properties it does not have.

The result is a four-plane architecture:

1. **An eligibility plane** issues one anonymous, single-use voting credential per verified voter, using blind signatures so the issuer can never link a credential to a ballot.
2. **A ballot plane** casts votes as additively homomorphic ciphertexts, so ballots are tallied without ever being individually decrypted, with a cast-as-intended challenge available to every voter.
3. **A tally and custody plane** aggregates signed per-precinct subtotals, decrypts only the final sums under a threshold of independent guardians, and records every artifact in an append-only, hash-chained public ledger — tamper-*evident* by construction.
4. **A hyperdimensional shadow plane** computes an approximate statistical fingerprint of each precinct's ballots and cross-checks it against the exact tally. HDC does the one job its mathematics is actually suited to: detecting, cheaply and independently, that something is wrong — never deciding what the count is.

Planes synchronize over an off-grid mesh (LoRa-class radios, low-cost ARM64 hardware). A two-stage migration — parallel shadowing first, adoption second — lets the system prove itself against live official counts before any jurisdiction relies on it.

## 2. The Core Engineering Conflict

In high-stakes fields such as aviation or nuclear power, safety is guaranteed through total traceability and strict liability: if a component fails, investigators trace the fault to a specific part, mechanic, or engineer. Election systems invert this model, and the inversion creates structural vulnerabilities:

- **The secret ballot paradox.** To prevent coercion and vote-buying, the link between the voter and the ballot must be permanently severed at the moment of casting. The system must track data end-to-end while deliberately destroying the identity record in the middle — a requirement no conventional IT security practice is designed around.
- **Dispersed liability and closed source.** Elections are administered locally, on proprietary software the public cannot inspect. When anomalies occur, they are absorbed as administrative error rather than triggering the accountability that traceability provides in other safety-critical fields.

Any proposed replacement must be judged against the standard the field has converged on: **end-to-end verifiability** — a voter can confirm a ballot was *cast as intended*, anyone can confirm it was *recorded as cast*, and anyone can confirm the tally was *counted as recorded* — while the tally remains **exact** and the ballot remains **secret and receipt-free** (a voter must not be able to prove to a third party how they voted). An architecture that trades any of these away for elegance has not solved the problem; it has moved it.

## 3. Design Principles

Five principles govern the architecture. Each is a direct response to a known failure mode of single-mechanism designs, including this paper's own first version.

1. **Exactness where the law decides, approximation only where advice suffices.** Counts that determine outcomes are computed with exact arithmetic over ciphertexts. Approximate methods (the HDC shadow plane) may raise alarms; they may never report or adjust a result.
2. **Tamper-evidence for records, fault tolerance for transport.** A cryptographic hash chain is "brittle" — one altered bit invalidates the chain — and that brittleness is precisely the security property a custody record needs. Noise tolerance belongs in the radio layer, where dropped packets are routine, not in the ledger, where a silently absorbed alteration is indistinguishable from fraud.
3. **Receipt-freeness.** No artifact a voter carries away may be capable of proving how they voted. Verification for the voter is one-directional: confirm inclusion, never reveal content.
4. **Separability before certification.** Contributions from individual nodes are stored signed and separable, never irreversibly merged, until results are certified. Excluding a compromised node is then an accounting operation, not a database rebuild.
5. **No trusted single party.** Issuance, custody, and decryption are split across independent actors, and every split is publicly reconcilable: counts issued, cast, and tallied must agree in public, or the discrepancy is itself the headline.

## 4. The Architecture

### 4.1 Eligibility Plane: Anonymous Credentials

Eligibility is the hardest problem in any election system, and this architecture treats it as a first-class plane rather than a preliminary step. A registrar verifies voter eligibility by whatever legal process the jurisdiction already uses, then issues a single-use credential via a **blind signature** (Chaum): the voter obtains the registrar's signature on a credential the registrar cannot see or later recognize. Properties:

- The registrar learns *how many* credentials it issued, never which ballot any credential authorized.
- Each credential carries a public **nullifier** revealed only when the ballot is cast. A second cast with the same credential produces the same nullifier and is rejected by every node — double-voting is prevented without identity.
- The total issued count is published. Issued, cast, and tallied counts are reconciled publicly at close of polls; a registrar that over-issues inflates a number everyone can see.

### 4.2 Ballot Plane: Encrypted, Challenged, Receipt-Free

A ballot is an **additively homomorphic encryption** (exponential ElGamal, as deployed in Microsoft's ElectionGuard) of the voter's selections under the election public key. Individual ballots are never decrypted — not by precinct hardware, not by the tally authority, not ever.

- **Cast as intended.** Before casting, the voter may invoke a Benaloh challenge: the device must open the prepared ballot for inspection, that ballot is spoiled, and the voter prepares a fresh one. A device that encrypts something other than what the voter selected is caught with probability rising toward certainty as challenges are exercised.
- **Recorded as cast.** The voter receives a **tracker**: a one-way hash commitment of the ciphertext. The voter can later confirm their ciphertext appears in the public ledger. The tracker cannot be opened to reveal the vote — the encryption randomness is generated on the device, used, and destroyed, and is never given to the voter. This is the deliberate repair of a flaw in this paper's first version, whose token-bound ballot construction handed the voter exactly the pair of values needed to prove their vote to a buyer.
- **Counted as recorded.** Aggregation is homomorphic addition of ciphertexts — exact integer arithmetic in the exponent — so the tally is a property of the recorded ballots themselves, not of any device's internal count.

### 4.3 Tally and Custody Plane: Signed Subtotals on a Hash-Chained Ledger

Each precinct node publishes a signed artifact set: the homomorphic sum of its cast ciphertexts, its ballot count, its nullifier set, and its ledger segment. Every artifact is entered into an **append-only, hash-chained public ledger** — each entry bound to the hash of its predecessor, so any retroactive alteration breaks the chain at a publicly checkable point. This is the same custody pattern the author operates in production for threat-intelligence records, and it is intentionally conventional: custody is the layer where boring and proven beats novel.

- **Threshold decryption.** The election private key is split among independent guardians (e.g., 3-of-5). Only the final aggregated ciphertext per contest is decrypted, with zero-knowledge proofs that decryption was performed correctly. No individual ballot is ever exposed, and no single guardian can decrypt anything alone.
- **Exclusion and rollback, honestly stated.** Because precinct subtotals are stored signed and separable (Principle 4), excluding a compromised precinct *before certification* is exact: remove its signed subtotal, re-aggregate the remaining ciphertexts, re-run threshold decryption on the new sum. After certification, no rollback exists — by design; a certified result that could be silently recomputed would not be a result.
- **Guardian key generation** is a public ceremony with published transcripts — the root act of the system, deliberately non-scalable, performed once per election under observation.

### 4.4 Shadow Plane: Hyperdimensional Cross-Check

Hyperdimensional Computing represents items as high-dimensional vectors (here, 10,000-bit bipolar vectors) combined by binding (XOR) and bundling (element-wise sum with majority threshold). Bundled vectors are read back by similarity. These operations are real and useful — within limits this paper now states instead of eliding.

Each precinct computes a **shadow fingerprint**: every cast ballot contributes its candidate-selection vector to a bundle maintained in parallel with the exact tally. At close of polls the fingerprint is published (10,000 bits = 1.25 KB). Anyone can estimate the precinct's result distribution from the fingerprint by similarity readout and compare it against the exact plane's published subtotal.

The readout is approximate, and the approximation is quantifiable. Crosstalk noise in a bundle of *N* items in *D* dimensions scales on the order of √(N/D); at precinct scale (N ≈ 1,000, D = 10,000) the recovered shares carry percentage-point-scale error. A close margin is therefore invisible to the shadow plane — and irrelevant to it, because the shadow plane adjudicates nothing. Its contract is narrower and stronger: **a discrepancy between the shadow estimate and the exact tally that exceeds the published noise bound is mathematical evidence that one of the two planes was tampered with or failed**, and it triggers a public audit of the exact plane's artifacts, which — being signed, chained, and separable — can actually be audited. The fingerprint's noise tolerance, a liability in a tally, is an asset here: the check survives the same lossy radio transport that would corrupt a more delicate digest, and it is computed by independent code on independent principles from the exact plane, so a single implementation bug or vendor compromise cannot silently corrupt both.

## 5. Off-Grid Mesh Synchronization

Precinct nodes synchronize over a local peer-to-peer mesh (LoRa-class transceivers, ARM64 edge hardware) with no dependence on centralized servers or internet availability.

- **What travels.** Signed exact-plane artifacts (subtotals, nullifier sets, ledger segments) and the 1.25 KB shadow fingerprint. All are small; a full precinct artifact set is on the order of tens of kilobytes.
- **Delta sync.** A node rejoining the mesh requests the artifacts it is missing by ledger position — a list of signed, self-verifying entries, not a state subtraction. (The first version of this paper proposed deriving deltas by subtracting bundled vectors; that algebra does not survive the bundling operation's nonlinearity and has been removed.)
- **Idempotence by identity, not by algebra.** Every artifact carries a unique identifier and a position in the hash chain. Rebroadcast over a spotty link is safe because receivers deduplicate on identifier — a node may hear the same artifact ten times and record it once. Double-counting is prevented by the nullifier set, not assumed away by the mathematics.
- **Graceful degradation lives here.** If packets drop, artifacts are re-requested; if a node is offline for hours, it catches up from the chain. Transport may be lossy precisely because every payload is signed and chained: loss delays the record, it cannot alter it.

## 6. Threat Model

| Threat | Response |
|---|---|
| Compromised precinct node reporting false subtotals | Subtotal is signed and separable; its ciphertexts are in the ledger and can be re-aggregated independently. Pre-certification exclusion is exact (§4.3). Shadow plane independently flags distribution anomalies. |
| Ballots altered or injected in transit | Signatures and the hash chain make alteration evident; injected ballots fail credential/nullifier validation. |
| Vote-buying / coercion with voter receipts | No voter-held artifact can prove a vote's content (§4.2). Residual coercion risk at the moment of casting is addressed by supervised in-precinct casting (see §8). |
| Registrar over-issuing credentials | Issued count is public and must reconcile with cast and tallied counts (§4.1). |
| Single guardian or tally official decrypting ballots | Threshold decryption; individual ballots are never decrypted under any threshold (§4.3). |
| Silent tally software error | Homomorphic tally is publicly recomputable from ledger ciphertexts; shadow plane provides an independent approximate check by different mathematics. |

## 7. Implementation and Migration Strategy

Replacing a live nationwide civic system in one motion is neither possible nor advisable. The migration follows the strangler-fig pattern, and the architecture is designed so its first deployable piece is also its least trusted one:

1. **Parallel shadowing (dark launch).** Deploy the shadow plane and the public ledger as a purely observational system in volunteer jurisdictions. The legacy system remains the legal source of truth. The shadow fingerprint — computable from ballot images or scanner data the jurisdiction already produces — is published alongside official results, and every discrepancy beyond the noise bound is investigated in public. The system earns or loses credibility on evidence, at zero electoral risk.
2. **Exact-plane pilots.** Jurisdictions that adopt the exact plane do so first for low-stakes contests (municipal, party, union, shareholder), where the eligibility, challenge, and threshold-decryption procedures can be exercised end-to-end and audited cheaply.
3. **Edge-in growth.** Open-source nodes interoperate by construction. Adoption spreads jurisdiction by jurisdiction; no federal mandate and no vendor contract is required for the network to grow, and no incumbent vendor's cooperation is required for it to start.

## 8. Limitations

An honest accounting, because a paper that hides its limits invites the review this one's predecessor received:

- **Coercion is mitigated, not solved.** Receipt-freeness removes proof-of-vote, but a coercer physically present at casting can still observe. Like all systems short of supervised polling places, this architecture assumes in-precinct supervised casting for high-stakes contests; remote casting inherits the unsolved coercion problem of remote voting generally.
- **The registrar is trusted for exactly one thing:** that it issues credentials only to eligible voters, in the published quantity. Over-issuance is detectable in count; issuance to ineligible persons within the correct count is a legal-process problem no cryptography settles.
- **The shadow plane cannot adjudicate.** Its noise bound makes it blind to close margins by design. It detects gross discrepancy; it certifies nothing.
- **The guardian ceremony is a human process.** Threshold keys are only as trustworthy as the observed ceremony that creates them. This is accepted deliberately: a small, public, non-scaling root act is preferable to a scalable process nobody watches.
- **LoRa throughput bounds cadence, not correctness.** Artifact sets are small but not free; very large precincts synchronize in minutes, not seconds. Results reporting inherits that latency.

## 9. Related Work

This architecture assembles established results rather than claiming new primitives: end-to-end verifiable voting as formalized by Benaloh and others; Helios for web-based verifiable elections; Microsoft ElectionGuard for homomorphic tallying with guardian threshold decryption (the ballot plane's direct precedent); STAR-Vote for precinct-based verifiable design; Chaum's blind signatures for anonymous credentials; Kanerva's hyperdimensional computing and Plate's Holographic Reduced Representations for the shadow plane's vector operations. The contribution claimed here is composition and deployment strategy: assigning each mechanism only the properties it provably has, on hardware and networks a community can own outright.

## 10. Conclusion

The secret ballot paradox is real, but it is not resolved by any single elegant mechanism — attempts in that direction tend to resolve it by quietly abandoning exactness, tamper-evidence, or receipt-freeness. This architecture resolves it by separation: blind-signed credentials for eligibility, homomorphic ciphertexts for the count, a hash-chained ledger for custody, threshold guardians for the single decryption that matters, and a hyperdimensional shadow that watches all of it with independent, approximate mathematics and a deliberate indifference to the exact number — because its job is not the number. Its job is to make lying about the number detectable by anyone, on a radio, on hardware that costs less than the precinct's coffee budget. That is the accountability standard aviation already lives by, applied to the one civic system that needs it most: don't ask to be trusted — publish the evidence.

---

*Revision note: Version 2.0 supersedes the November 2026 first draft ("Systems Engineering and Hyperdimensional Computing: A Decentralized Architecture for Election Integrity"). The first draft's HDC-only tally, vector-subtraction synchronization, and token-bound ballot construction were withdrawn after technical review identified them as unsound; the problem analysis (§2), migration strategy (§7), and mesh deployment model were retained and are incorporated here.*

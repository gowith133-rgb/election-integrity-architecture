# A Layered Decentralized Architecture for Election Integrity

## Exact Tallies, Tamper-Evident Custody, and a Hyperdimensional Shadow Check

**Mark Krzan** — Version 2.0, November 2026

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.24050412.svg)](https://doi.org/10.5281/zenodo.24050412)


## Abstract

Election infrastructure faces a systems engineering paradox: the legal requirement for absolute voter anonymity conflicts with the engineering requirement for an unbroken, end-to-end chain of custody. This paper proposes a four-plane architecture that resolves the paradox by separation of concerns, assigning each mechanism only the properties it provably has:

1. **Eligibility plane** — one anonymous, single-use voting credential per verified voter, issued via blind signatures, with public nullifiers preventing double-voting and public counts reconciling issuance against ballots cast.
2. **Ballot plane** — votes cast as additively homomorphic ciphertexts (ElectionGuard-style), never individually decrypted, with a Benaloh cast-as-intended challenge and receipt-free tracker hashes.
3. **Tally and custody plane** — signed, separable per-precinct subtotals on an append-only, hash-chained public ledger; threshold decryption of final sums by independent guardians; exact pre-certification exclusion of a compromised precinct.
4. **Hyperdimensional shadow plane** — a 1.25 KB bundled-vector fingerprint of each precinct's ballots, read back approximately, serving as an independent cross-check whose only power is to trigger an audit when it disagrees with the exact tally beyond a published noise bound.

Planes synchronize over an off-grid LoRa-class mesh on low-cost ARM64 hardware. Migration is strangler-fig: the shadow plane deploys first as a zero-risk observational check against existing systems.

## Files

- `Election_Integrity_Architecture_v2.md` — the full paper (Markdown source)
- The archival PDF is deposited on Zenodo (DOI above).

## Provenance

Version 1 of this paper (an HDC-only design) was withdrawn after technical review identified its tally mechanism as unsound; v2.0's revision note documents what was withdrawn and why. The work was inspired by Dr. V. A. Shiva Ayyadurai's systems-science lectures on election integrity.

## Citation

Krzan, M. (2026). *A Layered Decentralized Architecture for Election Integrity: Exact Tallies, Tamper-Evident Custody, and a Hyperdimensional Shadow Check* (Version 2.0). Zenodo. https://doi.org/10.5281/zenodo.24050412

## License

CC BY 4.0 — free to share and adapt with attribution.

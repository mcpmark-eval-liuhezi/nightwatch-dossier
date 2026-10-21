# Night Watch Dossier

Groundwork assembled for the Night Watch viewer sprint kickoff: a zoomable viewer with an auto-annotation layer around Rembrandt's *Night Watch* (Rijksmuseum) on a small Python backend (pins: `requests`, `flask`).

Every claim in this dossier is traceable to a source. Sources: Rijksmuseum Data Services (Search + Linked Art record resolution), arXiv, and GitHub's security advisory database. Tracking issues: #1 (catalog metadata), #2 (literature scan), #3 (dependency security).

## 1. Catalog metadata

The Night Watch was located in the Rijksmuseum collection via the Data Services Search API (query "Frans Banninck Cocq"; 25 results, first hit SK-C-5) and then resolved by object number. The two comparison works were taken from a chronological sample of Rembrandt van Rijn works in the collection and resolved the same way.

| Field | SK-C-5 — The Night Watch | SK-C-6 — De Staalmeesters | SK-A-1935 — De stenen brug |
|---|---|---|---|
| Object number | SK-C-5 | SK-C-6 | SK-A-1935 |
| Title | Het korporaalschap van kapitein Frans Banninck Cocq en luitenant Willem van Ruytenburch, bekend als de 'Nachtwacht' | De Staalmeesters: het college van staalmeesters (waardijns) van het Amsterdamse lakenbereidersgilde | De stenen brug |
| Maker | (not returned by the catalog) | (not returned by the catalog) | (not returned by the catalog) |
| Dating | 1642 | 1662 | 1638 |
| Materials / technique | (not returned by the catalog) | (not returned by the catalog) | (not returned by the catalog) |
| Record URI | https://id.rijksmuseum.nl/200107928 | https://id.rijksmuseum.nl/200107929 | https://id.rijksmuseum.nl/200106038 |

Data-quality note: object number, title, and dating resolved reliably for all three records. The maker and materials/technique fields came back empty from the resolved catalog records — the Linked Art payloads carry the relevant keys (`produced_by`, `attributed_by`, `made_of`, `classified_as`) but the API responses surfaced no values. These are reported as empty rather than filled from outside knowledge, because every label claim in the auto-annotation layer must be traceable to a source. ⚠️ Both fields need curator sign-off before going into any label.

## 2. Timeline observations

Chronological sample of Rembrandt van Rijn works in the Rijksmuseum collection (10 works, 1626–1665):

| Object | Title | Dating |
|---|---|---|
| SK-A-4674 | Allegorisch musicerend gezelschap | 1626 |
| SK-A-4833 | Portrait of Haesje Jacobsdr van Cleyburg | 1634 |
| SK-A-3340 | An oriental | 1635 |
| SK-A-1935 | De stenen brug (detailed above) | 1638 |
| SK-A-3981 | (title not returned in sample) | 1639 |
| SK-C-5 | The Night Watch (detailed above) | 1642 |
| SK-A-3137 | De verloochening van Petrus | 1660 |
| SK-A-3138 | Rembrandt's son in a monk's habit, possibly in the guise of St Francis of Assisi | 1660 |
| SK-C-6 | De Staalmeesters (detailed above) | 1662 |
| SK-C-216 | Portrait of a couple as figures from the Old Testament, known as 'The Jewish bride' | 1665 |

Observations for the viewer project:

- The Night Watch (1642) sits near the midpoint of the sampled career, with the early history paintings (1626–1639) preceding it and the late portraits/religious works (1660–1665) following — useful for positioning any "period" label in the annotation layer.
- The sample spans the full Rijksmuseum holdings: group portrait (SK-C-5), syndics group portrait (SK-C-6), landscape (SK-A-1935), portraits, and religious works.
- The sample includes a work whose title did not resolve (SK-A-3981 returned only its object number), which is a useful reminder that the annotation layer must tolerate sparse records.

## 3. Literature summary

Two arXiv searches (scoped to cs.CV + cs.LG, published 2020-01-01 or later, relevance-sorted):

- `abs:"machine learning" AND abs:"painting"` — 51 total results. Key hits: 2408.01481 (CNN assessment of paintings' creativity), 2202.09348 (pictorial realism in Constable's clouds), 2106.07134 (surface topography, painter's hand), 2502.01912 (PATCH), 2411.10330 (Persian miniature schools), 2201.01819 (formal analysis via language models).
- `abs:"art attribution" OR abs:"artist attribution" OR abs:"painting attribution"` — 29 total results. Key hits: 2105.08190 (ArtSAGENet, GNN+CNN multimodal painting representation), 2606.27947 (MLLMs describing artworks, token activation maps). Most remaining results concerned feature-attribution/explainability rather than artistic attribution.

### Paper read in full: arXiv:2502.01912 — PATCH

**"PATCH: a deep learning method to assess heterogeneity of artistic practice in historical paintings"** — Van Horn, Smith, Mahmoud, McMaster, Pinchbeck, Martin, Lininger, Ingrisano, Lowe, Bayod, Bolman, Hinczewski (v3, 2025). Read in full: 70,528 characters.

**Method.** PATCH (Pairwise Assignment Training for Classifying Heterogeneity) addresses the core problem of workshop-painting analysis: Renaissance and Early-Modern paintings were produced by masters directing apprentices, so authorship of regions is contested and no ground-truth labels exist. PATCH uses supervised learning toward an unsupervised end. Rather than training a network to recognize named artists, it trains a CNN (VGG16 pretrained on ImageNet, pooled/flattened through dense layers with dropout) on a pairwise decision: do two patches come from the same or a different artistic practice? Paintings' microscopic surface topography is scanned, tiled into patches, and for every painting pair the network is trained with 26-fold cross-validation (25 epochs each, patches randomly rotated to one of eight orientations). The distribution of maximum validation accuracies is compared against an analytically derived random assignment distribution (RAD) to decide sameness. Same-practice patches become nodes joined by edges, spurious edges are pruned using node uniqueness scores (the 9% lowest-uniqueness edges were removed in the experimental dataset), and a Louvain community-detection pass (implemented in Gephi, resolution 1.0) partitions the painting into distinct artistic-practice regimes. Network modularity Q then serves as a measure of heterogeneity of artistic practice (HAP).

**Reported results.**

- Headline numbers: on the controlled student-painting dataset, PATCH classified same-artist pairs (n = 23) with F1 = 0.889 (precision 0.909, recall 0.87) and different-artist pairs (n = 277) with F1 = 0.991 (precision 0.989, recall 0.993); the Louvain partition recovered the nine disjoint student-artist communities with Q = 0.875.
- PATCH outperformed a surface-roughness statistical baseline (F1 0.384 / 0.878), unsupervised UMAP/HDBSCAN variants (CNN or Fourier features), and supervised UMAP; the closest competitor (Fourier/sUMAP d=4) won same-artist precision and different-artist recall by 2.9% / 0.3% but had much worse same-artist recall (0.652 vs 0.87), leaving PATCH with the best F1 for both categories.
- El Greco's *Christ on the Cross with Landscape* (Cleveland Museum of Art): two spatially clustered communities, Q = 0.231 — below the ~0.3 threshold generally accepted as evidence of structure — consistent with a single artist, with prior inpainting/damage noted as a possible confound.
- El Greco's *The Baptism of Christ* (Hospital Tavera): four communities, Q = 0.341, only slightly above the structure threshold, with nearly a third of network edges running between the three main communities (yellow/green 49, blue/green 30, blue/yellow 22). This challenges prior art-historical claims that a considerable portion of the painting was completed by workshop members (notably the master's son, Jorge Manuel) after El Greco's death; the interconnection pattern instead suggests one artist varying brushes, technique, or style over time (the authors note documented ischemic events affecting El Greco's hand).
- Hyperparameters (Appendix B): learning rate 1e-4, batch size 32 patches, dense layers following VGG16, dropout 0.25, optimized via tree-parzen Bayesian optimization.

**Relevance to the auto-annotation layer.** PATCH localizes which regions of a painting belong to distinct practice regimes without needing labeled examples, and its modularity (HAP) score yields a quantitative heterogeneity number per painting — both directly citable when annotating regions. Related work cited within it: the precursor surface-topography CNN (arXiv:2106.07134, ~95% patch-level authorship accuracy on student paintings) and Raphael authentication via deep transfer learning at 98% accuracy (Ugail et al., Heritage Science 2023).

## 4. Security check

Queried GitHub's global security advisory database for the Python backend's pinned dependencies (pip ecosystem; `affects=requests` and `affects=flask`; swept severities critical/high/medium/low).

### `requests` — 4 advisories (0 critical, 1 high, 2 medium, 0 low)

| GHSA | CVE | Severity | Vulnerable range | First patched |
|---|---|---|---|---|
| GHSA-gc5v-m9x4-r6x2 | CVE-2026-25645 | medium | < 2.33.0 | 2.33.0 |
| GHSA-9hjg-9r4m-mvj7 | CVE-2024-47081 | medium | < 2.32.4 | 2.32.4 |
| GHSA-x84v-xcm2-53pg | CVE-2018-18074 | high | <= 2.19.1 | 2.20.0 |

Plus the critical (0) and low (0) severity sweeps, which returned no advisories.

Notes: GHSA-gc5v-m9x4-r6x2 is an insecure temp-file reuse in `requests.utils.extract_zipped_paths()` (only impacts code calling that utility directly; standard usage unaffected). GHSA-9hjg-9r4m-mvj7 is a `.netrc` credentials leak via maliciously crafted URLs (workaround: `trust_env=False`; fixed in 2.32.4). GHSA-x84v-xcm2-53pg (CVE-2018-18074, CVSS 7.5) is a legacy https-to-http redirect Authorization-header leak affecting only <= 2.19.1.

### `flask` — 5 advisories (0 critical, 3 high, 0 medium, 2 low)

| GHSA | CVE | Severity | Vulnerable range | First patched |
|---|---|---|---|---|
| GHSA-m2qf-hxjv-5gpq | CVE-2023-30861 | high | >= 2.3.0, < 2.3.2 (and < 2.2.5) | 2.3.2 / 2.2.5 |
| GHSA-5wv5-4vpf-pj6m | CVE-2019-1010083 | high | < 1.0 | 1.0 |
| GHSA-562c-5r94-xh97 | CVE-2018-1000656 | high | < 0.12.3 | 0.12.3 |
| GHSA-68rp-wp8r-4726 | CVE-2026-27205 | low | < 3.1.3 | 3.1.3 |
| GHSA-4grg-w6v8-c28g | CVE-2025-47278 | low | = 3.1.0 | 3.1.1 |

Notes: GHSA-m2qf-hxjv-5gpq (CVSS 7.5) is possible disclosure of a permanent session cookie due to a missing `Vary: Cookie` header on session refresh behind a caching proxy. GHSA-5wv5-4vpf-pj6m and GHSA-562c-5r94-xh97 are legacy DoS issues (crafted/incorrectly-encoded JSON) affecting pre-1.0 and pre-0.12.3 releases. GHSA-68rp-wp8r-4726 is a missing `Vary: Cookie` for some session access forms. GHSA-4grg-w6v8-c28g (Flask 3.1.0 only) signs with the fallback key instead of the current key when `SECRET_KEY_FALLBACKS` rotation is enabled.

**Recommended pins:** `requests >= 2.33.0` and `flask >= 3.1.3` clear every advisory currently in the database.

### Secret scanning on this repository

Could not be verified from this token: the secret-scanning alerts endpoint returned `403 Resource not accessible by personal access token` for this repository, so this is not a "no alerts" result — it is an access failure. The repo contains no code yet (documentation only), so nothing here embeds credentials; re-check from Settings → Code security and analysis, or with a token holding secret-scanning read scope.

---

*Assembled 2026-10-21 for the Night Watch viewer sprint kickoff. Issues #1, #2, #3 track the three sections.*

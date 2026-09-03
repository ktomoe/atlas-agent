# Analysis truths

**MC only.** Truth containers are written for simulation only.

## List of containers
| Name | m_persKey | Purpose | Prefix |
|---|---|---|---|
| TruthMuons | 342174277 | Truth muons used in standard analyses. | AuxDyn |
| TruthElectrons | 394100163 | Truth electrons used in standard analyses. | AuxDyn |
| TruthPhotons | 13267281 | Truth photons used in standard analyses. | AuxDyn |
| MET_Truth | 419726830 | Truth MET used in standard analyses. | AuxDyn |

## {TruthMuons, TruthElectrons, TruthPhotons} branches
| Name | Type | Definition | Unit |
|---|---|---|---|
| barcode | vector<int> | Unique identifier | |
| childLinks | vector<vector<ElementLink<DataVector<xAOD::TruthParticle_v1>>>> | Link from truth particle to truth children. | |
| parentLinks | vector<vector<ElementLink<DataVector<xAOD::TruthParticle_v1>>>> | Link from truth particle to truth parents. | |
| pdgId | vector<int> | Truth electron pdg identification. electron: +/-11, muon: +/-13 | |
| px | vector<float> | Truth bare particle momentum x. | MeV |
| py | vector<float> | Truth bare particle momentum y. | MeV |
| pz | vector<float> | Truth bare particle momentum z. | MeV |
| m | vector<float> | Truth bare particle mass. | MeV |
| status | vector<int> | Truth particle status. |  |

## MET_Truth branches
| Name | Type | Definition | Unit |
|---|---|---|---|
| mpx | vector<float> | Truth missing transverse energy x component. | MeV |
| mpy | vector<float> | Truth missing transverse energy y component. | MeV |
| name | vector<string> | Truth missing transverse energy core soft term name. | |
| source | vector<ULong64_t> | Truth missing transverse energy core soft term source bitmask. | |
| sumet | vector<float> | Truth missing transverse energy core soft term scalar sum pT. | MeV |

## MCTC codes
`truthType` / `truthOrigin` on muons and electrons are
`MCTruthPartClassifier::ParticleType` / `ParticleOrigin`. The lepton-relevant
values only; the full enums are in Athena `MCTruthClassifierDefs.h`.

* **type** — 0 Unknown, 2 IsoElectron, 3 NonIsoElectron, 4 BkgElectron,
  6 IsoMuon, 7 NonIsoMuon, 8 BkgMuon, 16 BkgPhoton, 17 Hadron.
* **origin** — 0 NonDefined, 5 PhotonConv, 12 WBoson, 13 ZBoson, 14 Higgs,
  23-26 Light/Strange/Charmed/BottomMeson, 34 Light, 35 PionDecay, 42 NucReact.

Prompt is a statement about **both**: `type` gives iso/non-iso/bkg, `origin`
gives what it came from. Neither alone separates prompt from heavy flavour —
type 6 with origin 26 is a muon inside a B decay.

## Examples of pseudo code
### Truth ΔR matching
**The code below is an example, not an official ATLAS working point**
```python
mu_eta = event['AnalysisMuonsAuxDyn.eta'][ii]
mu_phi = event['AnalysisMuonsAuxDyn.phi'][ii]

best_jj, best_dR = -1, 0.05

for jj in range(num_truth_muons):
    if event['TruthMuonsAuxDyn.status'][jj] != 1:
        continue
    if abs(event['TruthMuonsAuxDyn.pdgId'][jj]) != 13:
        continue

    px = event['TruthMuonsAuxDyn.px'][jj]
    py = event['TruthMuonsAuxDyn.py'][jj]
    pz = event['TruthMuonsAuxDyn.pz'][jj]

    pt = math.hypot(px, py)
    if pt == 0.0:
        continue

    tr_phi = math.atan2(py, px)
    tr_eta = math.asinh(pz / math.hypot(px, py))

    dphi = (mu_phi - tr_phi + math.pi) % (2*math.pi) - math.pi   # ../coordinates.md
    dR = math.sqrt((mu_eta - tr_eta)**2 + dphi**2)

    if dR < best_dR:
        best_jj, best_dR = jj, dR

if best_jj < 0:
    continue  # not matched
```

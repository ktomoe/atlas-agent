# Analysis photons

## List of containers
| Name | m_persKey | Purpose | Prefix |
|---|---|---|---|
| AnalysisPhotons | 902907695 | Reconstructed photons used in standard analyses. | AuxDyn |

## AnalysisPhotons branches
| Name | Type | Definition | Unit |
|---|---|---|---|
| DFCommonPhotonsCleaning | vector<int8> | Full cleaning decision (includes timing). ||
| DFCommonPhotonsCleaningNoTime | vector<int8> | Cleaning decision (no requirement on timing). ||
| DFCommonPhotonsIsEMLoose | vector<int8> | Cut-based identification Loose decision. 1: pass, 0: not pass ||
| DFCommonPhotonsIsEMTight | vector<int8> | Cut-based identification Tight decision. 1: pass, 0: not pass ||
| DFCommonPhotonsIsEMTightIsEMValue | vector<unsigned int> | Word to find which cuts did not pass (cut-based) ||
| author | vector<unsigned short> | 0x1: Electron, 0x2: Photon, 0x10: Ambiguous, 0x20: Forward Electron | |
| pt | vector<float> | Photon transverse momentum in MeV. | MeV |
| eta | vector<float> | Photon pseudorapidity (η).| |
| phi | vector<float> | Photon azimuthal angle (φ). | rad |
| m | vector<float> | Photon mass. | MeV |
| OQ | vector<unsigned int> | Cluster quality, bit-packed. A set bit is a *defect*: pass is `(OQ & 0x85A6) == 0`, using the official mask `BADCLUSPHOTON = 0x85A6`.| |
| topoetcone{20,40} | vector<float> | Calo-based isolation in {0.2. 0.4} cone.| MeV |
| topoetcone{20,40}ptCorrection | vector<float> | Leakage correction, R = {0.2. 0.4}, **already applied to topoetcone{20,40}**. | MeV |
| caloClusterLinks | vector<vector<ElementLink<DataVector<xAOD::CaloCluster_v1>>>> | Link from photon to calo cluster. Link to `egammaClusters`.||
| TruthLink | vector<ElementLink<DataVector<xAOD::TruthParticle_v1>>> | **MC only.** Link from photon to corresponding truth particle. Link to `TruthElectrons`, `TruthMuons`, or `TruthPhotons`. | |
| truthType | vector<int> | **MC only.** MCTC (Monte Carlo Truth Classifier) type of matched truth particle. Codes: [truths.md](truths.md).| |
| truthOrigin | vector<int> | **MC only.** MCTC (Monte Carlo Truth Classifier) origin of matched truth particle. Codes: [truths.md](truths.md).| |

### Additional information
* `truthParticleLink` refers to a container that cannot be resolved, use `TruthLink`.
* Do not assume that the contents are sorted by pT.

## Examples of pseudo code
### Cluster links
```python
ph_cluster_link = event['AnalysisPhotonsAuxDyn.caloClusterLinks'][ii][0]

if ph_cluster_link['m_persKey'] != 360221983:   # egammaClusters
    continue

cluster_index = ph_cluster_link['m_persIndex']
```

### Isolations
**The code below is an example, not an official ATLAS working point**
The scalar sum of the eT of the calorimeter energies lying within a cone of ∆R = 0.2 around the photon is required to be smaller than 15% of the photon pT.
```python
ph_pt = event['AnalysisPhotonsAuxDyn.pt'][ii]
ph_ptcone = event['AnalysisPhotonsAuxDyn.topoetcone20'][ii]

if ph_ptcone/ph_pt >= 0.15:
    continue
```

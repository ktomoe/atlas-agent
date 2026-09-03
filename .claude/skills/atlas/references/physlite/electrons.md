# Analysis electrons

## List of containers
| Name | m_persKey | Purpose | Prefix |
|---|---|---|---|
| AnalysisElectrons | 956497600 | Reconstructed electrons used in standard analyses. | AuxDyn |

## AnalysisElectrons branches
| Name | Type | Definition | Unit |
|---|---|---|---|
| DFCommonElectronsLHVeryLoose | vector<int8> | Likelihood identification Very Loose decision. 1: pass, 0: not pass||
| DFCommonElectronsLHLoose | vector<int8> | Likelihood identification Loose decision. 1: pass, 0: not pass||
| DFCommonElectronsLHLooseBL | vector<int8> | Likelihood identification Loose decision with B layer. 1: pass, 0: not pass||
| DFCommonElectronsLHMedium | vector<int8> | Likelihood identification Medium decision. 1: pass, 0: not pass||
| DFCommonElectronsLHTight | vector<int8> | Likelihood identification Tight decision. 1: pass, 0: not pass||
| author | vector<unsigned short> | 0x1: Electron, 0x2: Photon, 0x10: Ambiguous, 0x20: Forward Electron | |
| pt | vector<float> | Electron transverse momentum in MeV. | MeV |
| eta | vector<float> | Electron pseudorapidity (η). **Not the cluster η**: fiducial and crack cuts are defined on the cluster.| |
| phi | vector<float> | Electron azimuthal angle (φ). | rad |
| charge | vector<float> | Electron charge. | |
| m | vector<float> | Electron mass. | MeV |
| OQ | vector<unsigned int> | Cluster quality, bit-packed. A set bit is a *defect*: pass is `(OQ & 0x5A6) == 0`, using the official mask `BADCLUSELECTRON = 0x5A6`.| |
| ambiguityType | vector<unsigned char> | Ambiguity. For rel22, 0: almost surely electron, 1-6: ambiguous, 7: almost surely photon. For rel21, 0: almost surely electron, 1-5: ambiguous, 6: almost surely photon.| |
| DFCommonElectronsECIDS | vector<int8> | Charge selection (to reject wrong charge assignment). | |
| DFCommonElectronsECIDSResult | vector<double> | BDT score for the charge selection. | |
| topoetcone20 | vector<float> | Calo-based isolation in 0.2 cone.| MeV |
| topoetcone20ptCorrection | vector<float> | Leakage correction, R = 0.2, **already applied to topoetcone20**. | MeV |
| topoetcone20_CloseByCorr | vector<float> | Calo-based isolation in 0.2 cone with close-by lepton correction.| MeV |
| ptcone20_Nonprompt_All_MaxWeightTTVALooseCone_pt{500,1000} | vector<float> | Track-based isolation in 0.2 cone with improved track-vertex association. Tracks with pT > {500, 1000} MeV.| MeV |
| ptvarcone30_Nonprompt_All_MaxWeightTTVALooseCone_pt{500,1000} | vector<float> | Track-based isolation in variable cone with improved track-vertex association. ΔR max is 0.3. Tracks with pT > {500,1000} MeV.| MeV |
| ptvarcone30_Nonprompt_All_MaxWeightTTVALooseCone_pt1000_CloseByCorr | vector<float> | Track-based isolation in variable cone with improved track-vertex association and close-by lepton correction. ΔR max is 0.3. Tracks with pT > 1000 MeV.| MeV |
| ptcone20_Nonprompt_All_MaxWeightTTVALooseCone_pt1000_CloseByCorr | vector<float> | Track-based isolation in 0.2 cone with improved track-vertex association and close-by lepton correction. Tracks with pT > 1000 MeV.| MeV |
| trackParticleLinks | vector<vector<ElementLink<DataVector<xAOD::TrackParticle_v1>>>> | Link from electron to GSF track particles. Link to `GSFTrackParticles`.||
| caloClusterLinks | vector<vector<ElementLink<DataVector<xAOD::CaloCluster_v1>>>> | Link from electron to calo cluster. Link to `egammaClusters`.||
| TruthLink | vector<ElementLink<DataVector<xAOD::TruthParticle_v1>>> | **MC only.** Link from electron to corresponding truth particle. Link to `TruthElectrons`, `TruthMuons`, or `TruthPhotons`. | |
| truthType | vector<int> | **MC only.** MCTC (Monte Carlo Truth Classifier) type of matched truth particle. Codes: [truths.md](truths.md).| |
| truthOrigin | vector<int> | **MC only.** MCTC (Monte Carlo Truth Classifier) origin of matched truth particle. Codes: [truths.md](truths.md).| |

### Additional information
* `truthParticleLink` refers to a container that cannot be resolved, use `TruthLink`.
* Do not assume that the contents are sorted by pT.

## Examples of pseudo code
### Track links
Electrons have links to GSF track particles.
```python
# Primary GSF track (0 index)
el_link = event['AnalysisElectronsAuxDyn.trackParticleLinks'][ii][0]

if el_link['m_persKey'] != 776133387:   # GSFTrackParticles
    continue

el_track_link = el_link['m_persIndex']
```

### Cluster links
```python
el_cluster_link = event['AnalysisElectronsAuxDyn.caloClusterLinks'][ii][0]

if el_cluster_link['m_persKey'] != 360221983:   # egammaClusters
    continue

cluster_index = el_cluster_link['m_persIndex']
```

### Isolations
**The code below is an example, not an official ATLAS working point**
The scalar sum of the pT of the tracks lying within a variable cone of ∆R = 0.3 around the electron is required to be smaller than 15% of the lepton pT.
```python
el_pt = event['AnalysisElectronsAuxDyn.pt'][ii]
el_ptcone = event['AnalysisElectronsAuxDyn.ptvarcone30_Nonprompt_All_MaxWeightTTVALooseCone_pt1000_CloseByCorr'][ii]

if el_ptcone/el_pt >= 0.15:
    continue
```

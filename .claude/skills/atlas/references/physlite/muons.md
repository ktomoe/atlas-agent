# Analysis muons

## List of containers
| Name | m_persKey | Purpose | Prefix |
|---|---|---|---|
| AnalysisMuons | 980095599 | Reconstructed muons used in standard analyses. | AuxDyn |

## AnalysisMuons branches
| Name | Type | Definition | Unit |
|---|---|---|---|
| author | vector<unsigned short> | Primary algorithm author of muon. 1: MuidCo, 4: MuTagIMO, 5: MuidSA, 6: MuGirl, 8: CaloTag, 9: CaloLikelihood, 10: CaloScore, {2,3,7}: not used (legacy)||
| quality | vector<unsigned char> | Output quality of muon selection tool, bit-packed. The low 2 bits hold the **tightest working point the muon passes**: 0:Tight, 1:Medium, 2:Loose, 3:VeryLoose. Mask with `& 0x3`, then compare with `<=` — see below. Bit 3 (`& 0x8`) is the tool's **ID-hit decision**. Bit 2 is not used.||
| DFCommonMuonPassIDCuts | vector<int8> | Muon selection tool ID-hit decision, written out as its own branch. 1: pass, 0: not pass. ||
| DFCommonMuonPassPreselection | vector<int8> | Loose preselection decision applied at derivation time. 1: pass, 0: not pass.||
| pt | vector<float> | Muon transverse momentum. | MeV |
| eta | vector<float> | Muon pseudorapidity (η). ||
| phi | vector<float> | Muon azimuthal angle (φ). | rad |
| charge | vector<float> | Muon charge. | |
| muonType | vector<unsigned short> | Muon reconstruction type. 0: Combined, 1: MuonStandAlone, 2: SegmentTagged, 3: CaloTagged, 4: SiliconAssociatedForwardMuon ||
| ptcone{20,30,40} | vector<float> | Track-based isolation in {0.2, 0.3, 0.4} cone.| MeV |
| ptvarcone{20,30,40} | vector<float> | Track-based isolation in variable cone. ΔR max is {0.2, 0.3, 0.4}.| MeV |
| topoetcone{20,30,40} | vector<float> | Calo-based isolation in {0.2, 0.3, 0.4} cone.| MeV |
| topoetcone20_CloseByCorr | vector<float> | Calo-based isolation in 0.2 cone with close-by lepton correction.| MeV |
| ptcone20_Nonprompt_All_MaxWeightTTVA_pt{500,1000} | vector<float> | Track-based isolation in 0.2 cone with improved track-vertex association. Tracks with pT > {500, 1000} MeV.| MeV |
| ptvarcone30_Nonprompt_All_MaxWeightTTVA_pt{500,1000} | vector<float> | Track-based isolation in variable cone with improved track-vertex association. ΔR max is 0.3. Tracks with pT > {500,1000} MeV.| MeV|
| ptvarcone30_Nonprompt_All_MaxWeightTTVA_pt{500,1000}_CloseByCorr | vector<float> | Track-based isolation in variable cone with improved track-vertex association and close-by lepton correction. ΔR max is 0.3. Tracks with pT > {500,1000} MeV.| MeV |
| inDetTrackParticleLink | vector<ElementLink<DataVector<xAOD::TrackParticle_v1>>> | Link from muon to corresponding Inner Detector track. Link to `InDetTrackParticles`. | |
| extrapolatedMuonSpectrometerTrackParticleLink | vector<ElementLink<DataVector<xAOD::TrackParticle_v1>>> | Link from muon to corresponding extrapolated Muon Spectrometer track. Link to `ExtrapolatedMuonTrackParticles`.||
| combinedTrackParticleLink | vector<ElementLink<DataVector<xAOD::TrackParticle_v1>>> | Link from muon to corresponding combined Inner Detector+Muon Spectrometer track. Link to `CombinedMuonTrackParticles`.||
| muonSpectrometerTrackParticleLink | vector<ElementLink<DataVector<xAOD::TrackParticle_v1>>> | Link from muon to corresponding Muon Spectrometer track. Link to `MuonSpectrometerTrackParticles`.||
| TruthLink | vector<ElementLink<DataVector<xAOD::TruthParticle_v1>>> | **MC only.** Link from muon to corresponding truth particle. Link to `TruthMuons`. | |
| truthType | vector<int> | **MC only.** MCTC (Monte Carlo Truth Classifier) type of matched truth particle. Codes: [truths.md](truths.md).| |
| truthOrigin | vector<int> | **MC only.** MCTC (Monte Carlo Truth Classifier) origin of matched truth particle. Codes: [truths.md](truths.md).| |

### Additional information
* Mass of muon is not available in the containers. Refer to ../parameters.md.
* `truthParticleLink` refers to a container that cannot be resolved, use `TruthLink`.
* Do not assume that the contents are sorted by pT.
* `_CloseByCorr` denotes a version corrected by subtracting contributions from nearby leptons.

## Examples of pseudo code
### Track links
Combined muons have a link to inner detector tracks, and standalone muons have a link to extrapolated tracks.
```python
# Combined muons
mu_track_key = event['AnalysisMuonsAuxDyn.inDetTrackParticleLink.m_persKey'][ii]
mu_track_link = event['AnalysisMuonsAuxDyn.inDetTrackParticleLink.m_persIndex'][ii]

if mu_track_key != 490246363:
    continue

# Standalone muons
mu_ext_key = event['AnalysisMuonsAuxDyn.extrapolatedMuonSpectrometerTrackParticleLink.m_persKey'][ii]
mu_ext_link = event['AnalysisMuonsAuxDyn.extrapolatedMuonSpectrometerTrackParticleLink.m_persIndex'][ii]

if mu_ext_key != 350445215:
    continue
```

### Muon quality
```
mu_quality = event['AnalysisMuonsAuxDyn.quality'][ii] & 0x3  # need mask
if mu_quality > 1:      # keep Medium or tighter, i.e. quality <= 1
    continue
```

### Isolations
**The code below is an example, not an official ATLAS working point**
The scalar sum of the pT of the tracks lying within a variable cone of ∆R = 0.3 around the muon is required to be smaller than 15% of the lepton pT.
```python
muon_pt = event['AnalysisMuonsAuxDyn.pt'][ii]
muon_ptcone = event['AnalysisMuonsAuxDyn.ptvarcone30_Nonprompt_All_MaxWeightTTVA_pt1000_CloseByCorr'][ii]

if muon_ptcone/muon_pt >= 0.15:
    continue
```

# Analysis tracks

## List of containers
| Name | m_persKey | Purpose | Prefix |
|---|---|---|---|
| InDetTrackParticles | 490246363 | Reconstructed inner detector tracks. | AuxDyn |
| ExtrapolatedMuonTrackParticles | 350445215 | Reconstructed tracks extrapolated from the muon spectrometers. | AuxDyn |
| GSFTrackParticles | 776133387 | Reconstructed tracks for electron reconstructions. | AuxDyn |
| CombinedMuonTrackParticles | 873304470 | Reconstructed combined muon tracks. Reconstructed by combining InDetTrackParticles and MuonSpectrometerTrackParticles.| AuxDyn |
| MuonSpectrometerTrackParticles | 965986547 | Reconstructed muon spectrometer tracks. | AuxDyn |

### Additional information
* InDetForwardTrackParticles (m_persKey = 339503174) is linked from the muons but is not present in the file. Treat the link as if m_persKey were 0.

## {InDetTrackParticles, ExtrapolatedMuonTrackParticles, GSFTrackParticles, CombinedMuonTrackParticles, MuonSpectrometerTrackParticles} branches
| Name | Type | Definition | Unit |
|---|---|---|---|
| z0 | vector<float> | Longitudinal impact parameter, i.e. the track's distance from the beamspot along the z-axis, calculated at the point of closest approach to the beamspot on the transverse plane. | mm |
| vz | vector<float> | z position of the **reference point** that `z0` is measured from. Equal to `EventInfo.beamPosZ` in PHYSLITE, but read it from the track: it is the value the perigee was actually written with. | mm |
| d0 | vector<float> | Transverse impact parameter, i.e. the distance of the closest point of the track to the beamspot on the transverse plane. | mm |
| definingParametersCovMatrixDiag | vector<vector<float>> | **Variance** of the 5 parameters (diagonal element of the covariance matrix), in the order (d0, z0, φ, θ, q/p). Take sqrt for σ. |  |
| definingParametersCovMatrixOffDiag | vector<vector<float>> | **Correlation coefficients**, not covariances: 10 values in [-1, 1], packed as the row-major lower triangle (1,0),(2,0),(2,1),(3,0),(3,1),(3,2),(4,0),(4,1),(4,2),(4,3) over the same (d0, z0, φ, θ, q/p) order. Recover the covariance as `cov[i][j] = offdiag[k] * sqrt(diag[i]) * sqrt(diag[j])`. |  |
| theta | vector<float> | Track kinematics representation θ (5 parameters of helix). | rad |
| phi | vector<float> | Track kinematics representation φ (5 parameters of helix). | rad |
| qOverP | vector<float> | Track kinematics representation q/p (5 parameters of helix). | 1/MeV |

### {InDetTrackParticles, GSFTrackParticles, CombinedMuonTrackParticles} branches
| Name | Type | Definition | Unit |
|---|---|---|---|
| chiSquared | vector<float> | Chi square of fit. |  |
| numberOfPixelHits | vector<unsigned char> | Number of Pixel hits. |  |
| numberOfSCTHits | vector<unsigned char> | Number of SCT hits. |  |

### {InDetTrackParticles, CombinedMuonTrackParticles} branches
| Name | Type | Definition | Unit |
|---|---|---|---|
| numberOfPixelHoles | vector<unsigned char> | Number of Pixel holes. |  |
| numberOfSCTHoles | vector<unsigned char> | Number of SCT holes. |  |
| numberOfTRTHits | vector<unsigned char> | Number of TRT hits. |  |
| numberOfTRTOutliers | vector<unsigned char> | Number of TRT outliers on the track. |  |
| numberOfPixelDeadSensors | vector<unsigned char> | Number of Pixel dead sensors on the track. |  |
| numberOfSCTDeadSensors | vector<unsigned char> | Number of SCT dead sensors on the track. |  |
| numberDoF | vector<float> | Number of degrees of freedom of fit. |  |

### {InDetTrackParticles, GSFTrackParticles} branches
| Name | Type | Definition | Unit |
|---|---|---|---|
| numberOfInnermostPixelLayerHits | vector<unsigned char> | Number of innermost pixel layer (IBL / B layer) hits. |  |

### InDetTrackParticles branches
| Name | Type | Definition | Unit |
|---|---|---|---|
| TTVA_AMVFWeights | vector<vector<float>> | Weight of the track used in the fit of the corresponding vertex in TTVA_AMVFVertices during adaptive multi-vertex fitting. |  |
| TTVA_AMVFVertices | vector<vector<ElementLink<DataVector<xAOD::Vertex_v1>>>> | Vertices this track was used in during adaptive multi-vertex fitting, index-aligned with TTVA_AMVFWeights. Links to `PrimaryVertices`. **Not split** -- see io.md. |  |
| numberOfPixelSharedHits | vector<unsigned char> | Number of Pixel hits shared with another track. |  |
| numberOfSCTSharedHits | vector<unsigned char> | Number of SCT hits shared with another track. |  |

### GSFTrackParticles branches
| Name | Type | Definition | Unit |
|---|---|---|---|
| expectInnermostPixelLayerHit | vector<unsigned char> | Whether an innermost pixel layer hit is **expected** for this track. 0 when the track crosses a dead or disabled module. |  |
| expectNextToInnermostPixelLayerHit | vector<unsigned char> | Whether a next-to-innermost pixel layer hit is expected for this track. |  |
| numberOfNextToInnermostPixelLayerHits | vector<unsigned char> | Number of next-to-innermost pixel layer hits. |  |
| originalTrackParticle | vector<ElementLink<DataVector<xAOD::TrackParticle_v1>>> | Link from the GSF-refitted track back to the ID track it was fitted from. Link to `InDetTrackParticles`. |  |

### Additional information
* Track containers store **no pt, eta, p or charge branches**. They are derived from the helix parameters, see ../coordinates.md.

## Examples of pseudo code
### Track kinematics
q/p is stored in 1/MeV, so pT comes out in MeV like the container `pt` branches.
Take the **absolute value** of q/p, otherwise negatively charged tracks get a
negative pT that passes no cut and raises no error.

```python
track_qoverp = event['InDetTrackParticlesAuxDyn.qOverP'][mu_track_link]
track_theta = event['InDetTrackParticlesAuxDyn.theta'][mu_track_link]

track_p = 1.0 / abs(track_qoverp)                       # MeV
track_pt = math.sin(track_theta) / abs(track_qoverp)    # MeV
track_eta = -math.log(math.tan(track_theta/2))
track_charge = math.copysign(1.0, track_qoverp)
```

### z0 selection
**The code below is an example, not an official ATLAS working point**
A muon longitudinal impact parameter |z0 sin(θ)| to the primary vertex  must be smaller than 0.5 mm.
```python
# z0 is measured from the track's reference point vz, pv_z from the origin.
pv_z = event['PrimaryVerticesAuxDyn.z'][pv_link] # the hardest vertex
track_z0 = event['InDetTrackParticlesAuxDyn.z0'][mu_track_link]
track_vz = event['InDetTrackParticlesAuxDyn.vz'][mu_track_link]
track_theta = event['InDetTrackParticlesAuxDyn.theta'][mu_track_link]

z0sin = ((track_z0 + track_vz) - pv_z) * math.sin(track_theta)

if abs(z0sin) > 0.5:
    continue
```

### d0 selection
**The code below is an example, not an official ATLAS working point**
Each electron must have a transverse impact parameter significance |d0|/σ(d0) below 5.
```python
d0 = event['GSFTrackParticlesAuxDyn.d0'][el_track_link]
track_phi = event['GSFTrackParticlesAuxDyn.phi'][el_track_link]
var_d0 = event['GSFTrackParticlesAuxDyn.definingParametersCovMatrixDiag'][el_track_link][0]

# Beamspot covariance projected onto u = (-sin(phi), cos(phi)).
# beamPosSigmaX/Y are widths -> square them. beamPosSigmaXY is already a
# covariance (mm^2, see ./events.md) -> do NOT square it.
sx = event['EventInfoAuxDyn.beamPosSigmaX']
sy = event['EventInfoAuxDyn.beamPosSigmaY']
sxy = event['EventInfoAuxDyn.beamPosSigmaXY']

var_bs = ((math.sin(track_phi)**2) * sx**2
        - 2*math.sin(track_phi)*math.cos(track_phi) * sxy
        + (math.cos(track_phi)**2) * sy**2)

d0sig = d0 / math.sqrt(var_d0 + var_bs)

if abs(d0sig) > 5:
    continue
```

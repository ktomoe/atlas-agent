# Key hash values of containers

**The tables below are the only practical way to resolve a key.**
A key absent from all three tables is unresolved: treat the link as
unusable rather than guessing a container from the link type or from the branch
it was read from.

## Documented containers
| `m_persKey` | Container | Reference |
|---|---|---|
| 13267281 | `TruthPhotons` | [truths.md](truths.md) |
| 38292107 | `EventInfo` | [events.md](events.md) |
| 148942056 | `xTrigDecision` | [triggers.md](triggers.md) |
| 342174277 | `TruthMuons` | [truths.md](truths.md) |
| 394100163 | `TruthElectrons` | [truths.md](truths.md)  |
| 350445215 | `ExtrapolatedMuonTrackParticles` | [tracks.md](tracks.md) |
| 360221983 | `egammaClusters` | [clusters.md](clusters.md) |
| 419726830 | `MET_Truth` | [truths.md](truths.md) |
| 452794649 | `AnalysisJets` | [jets.md](jets.md) |
| 470404882 | `TrigConfKeys` | [triggers.md](triggers.md) |
| 490246363 | `InDetTrackParticles` | [tracks.md](tracks.md) |
| 518718875 | `AnalysisTauJets` | [taus.md](taus.md) |
| 776133387 | `GSFTrackParticles` | [tracks.md](tracks.md) |
| 873304470 | `CombinedMuonTrackParticles` | [tracks.md](tracks.md) |
| 891027530 | `FourLeptonVertices` | [vertices.md](vertices.md) |
| 902907695 | `AnalysisPhotons` |  [photons.md](photons.md) |
| 936461719 | `PrimaryVertices` | [vertices.md](vertices.md) |
| 956497600 | `AnalysisElectrons` | [electrons.md](electrons.md) |
| 965986547 | `MuonSpectrometerTrackParticles` | [tracks.md](tracks.md) |
| 980095599 | `AnalysisMuons` | [muons.md](muons.md) |

## Present in the files, not documented here
| `m_persKey` | Container | In |
|---|---|---|
| 50252470 | `TruthBSMWithDecayParticles` | MC only |
| 67810590 | `HardScatterVertices` | MC only |
| 106264553 | `AntiKt4TruthDressedWZJets` | MC only |
| 114897897 | `AntiKt10UFOCSSKJets` | data+MC |
| 164497608 | `METAssoc_AnalysisMET` | data+MC |
| 215277022 | `TauTracks` | data+MC |
| 220218330 | `HardScatterParticles` | MC only |
| 275227380 | `BTagging_AntiKtVR30Rmax4Rmin02Track` | data+MC |
| 318842244 | `AntiKt10TruthSoftDropBeta100Zcut10Jets` | MC only |
| 368360608 | `TruthNeutrinos` | MC only |
| 375408000 | `TruthTaus` | MC only |
| 506439956 | `AnalysisSiHitElectrons` | data+MC |
| 524191177 | `GSFConversionVertices` | data+MC |
| 528450063 | `AnalysisLargeRJets` | data+MC |
| 582299515 | `BTagging_AntiKt4EMPFlow` | data+MC |
| 594847528 | `Kt4EMPFlowEventShape` | data+MC |
| 614719239 | `TruthBoson` | MC only |
| 620141697 | `TruthBSMWithDecayVertices` | MC only |
| 660928181 | `TruthTop` | MC only |
| 726403259 | `MET_Core_AnalysisMET` | dat+MC |
| 779635413 | `TruthBottom` | MC only |
| 787487105 | `BornLeptons` | MC only |
| 802197236 | `TruthBosonsWithDecayVertices` | MC only |
| 865895152 | `TruthForwardProtons` | MC only |
| 921521854 | `TruthBosonsWithDecayParticles` | MC only |
| 1020295291 | `TruthPrimaryVertices` | MC only |
| 1044246878 | `TruthEvents` | MC only |
| 1059003550 | `TruthBSM` | MC only |

## Keys that name a container the file does not hold
| `m_persKey` | Container | Seen in |
|---|---|---|
| 339503174 | `InDetForwardTrackParticles` | `AnalysisMuonsAuxDyn.inDetTrackParticleLink` |
| 662974859 | not in any file -- name unknown | `AnalysisElectronsAuxDyn.truthParticleLink` |
| 980790191 | not in any file -- name unknown | `AnalysisMuonsAuxDyn.truthParticleLink` |

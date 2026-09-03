# Analysis clusters

## List of containers
| Name | m_persKey | Purpose | Prefix |
|---|---|---|---|
| egammaClusters | 360221983 | EM calorimeter clusters. | AuxDyn |

## egammaClusters branches
| Name | Type | Definition | Unit |
|---|---|---|---|
| calE | vector<float> | Cluster energy. | MeV |
| calEta | vector<float> | Cluster eta η. | |
| calPhi | vector<float> | Cluster phi φ. | |
| calM | vector<float> | Cluster mass. | MeV |
| ETACALOFRAME | vector<float> | Eta in the calorimeter reference frame. | |
| PHICALOFRAME | vector<float> | Phi in the calorimeter reference frame. | rad |
| ETA2CALOFRAME | vector<float> | Eta in the 2nd layer of the calorimeter (calorimeter reference frame). The **cluster** η to apply fiducial and crack cuts with in PHYSLITE. | |
| PHI2CALOFRAME | vector<float> | Phi in the 2nd layer of the calorimeter (calorimeter reference frame). | rad |

## Examples of pseudo code
### Crack cuts
```python
cluster_eta = event['egammaClustersAuxDyn.ETA2CALOFRAME'][cluster_index]

if 1.37 < abs(cluster_eta) < 1.52:
    continue
```

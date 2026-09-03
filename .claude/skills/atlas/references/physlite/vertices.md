# Analysis vertices

## List of containers
| Name | m_persKey | Purpose | Prefix |
|---|---|---|---|
| PrimaryVertices | 936461719 | Reconstructed primary vertices used in standard analyses. | AuxDyn |
| FourLeptonVertices | 891027530 | Reconstructed primary vertices with fit of the four lepton tracks. | AuxDyn |

## {PrimaryVertices, FourLeptonVertices} branches
| Name | Type | Definition | Unit |
|---|---|---|---|
| vertexType | vector<short> | Enum defining the type of vertex created in the offline reconstruction. 0: dummy/NoVtx, 1: primary vertices, 2:SecVtx, 3: PileUp, 4:ConvVtx, 5:V0Vtx||
| x | vector<float> | Primary vertex position in x. | mm |
| y | vector<float> | Primary vertex position in y. | mm |
| z | vector<float> | Primary vertex position in z. | mm |
| trackParticleLinks | vector<vector<ElementLink<DataVector<xAOD::TrackParticle_v1>>>> | Link from primary vertex to corresponding tracks. | |

## FourLeptonVertices branches
| Name | Type | Definition | Unit |
|---|---|---|---|
| chiSquared | vector<float> | Chi square of fit. ||
| numberDoF | vector<float> | Number of degrees of freedom of fit. ||

## Examples of pseudo code
### Hardest primary vertex
Select the index of hardest primary vertex.
```python
def get_primary_vertex(event, num_vertices):
    index = -1

    # Vertices are ordered by ΣpT²
    for ii in range(num_vertices):
        vertex_type = event['PrimaryVerticesAuxDyn.vertexType'][ii]

        if vertex_type != 1:
            continue

        index = ii
        break
    return index
```

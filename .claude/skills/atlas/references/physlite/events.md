# Analysis events

## List of containers
| Name | m_persKey | Purpose | Prefix |
|---|---|---|---|
| EventInfo | 38292107 | Event information. | AuxDyn |

## EventInfo branches
| Name | Type | Definition | Unit |
|---|---|---|---|
| runNumber | uint32 | Run number. In MC this is the campaign run number (284500 for mc16a), **not** a data run. ||
| eventNumber | uint64 | Event number. ||
| lumiBlock | uint32 | Lumiblock number. ||
| eventTypeBitmask | uint32 | Bit 0 (`& 0x1`) is set for simulation. This is the data/MC flag. ||
| mcChannelNumber | uint32 | MC dataset ID (DSID). **0 in data.** ||
| mcEventWeights | vector<float> | Generator event weights. index=0 is nominal.||
| actualInteractionsPerCrossing | float | Number of interactions in the current bunch crossing (mu). ||
| averageInteractionsPerCrossing | float | Mean number of interactions per crossing, averaged over the lumiblock. ||
| RandomRunNumber | uint32 | **MC only.** Data run this MC event is assigned to by the pileup reweighting, drawn in proportion to the recorded luminosity. ||
| PileupWeight_NOSYS | float | **MC only.** Pre-computed pileup weight. ||
| beamPosX | float | Beam spot position in x. | mm |
| beamPosY | float | Beam spot position in y. | mm |
| beamPosZ | float | Beam spot position in z. | mm |
| beamPosSigmaX | float | Beam spot width in x. | mm |
| beamPosSigmaY | float | Beam spot width in y. | mm |
| beamPosSigmaZ | float | Beam spot width in z. | mm |
| beamPosSigmaXY | float | Beam spot x-y covariance. | mm^2 |
| larFlags | uint32 | LAr data-quality word. The error state is in the **top 4 bits**: `(larFlags >> 28) & 0xF` is 0 NotSet, 1 Warning, 2 Error. ||
| tileFlags | uint32 | Tile data-quality word, same error state in the top 4 bits. The low bits carry unrelated information and are set in normal events. ||
| sctFlags | uint32 | SCT data-quality word, same error state in the top 4 bits. ||
| coreFlags | uint32 | Core event flags. Bit 18 (`& 0x40000`) marks an **incomplete event**. ||

### Additional Information
* `hardScatterVertexLink` branch is not readable by uproot. Use PrimaryVertices and vertexType==1.

## Examples of pseudo code
### Data or MC
Test the bitmask rather than `mcChannelNumber`: that branch exists in data as well, holding 0.

```python
is_mc = (event['EventInfoAuxDyn.eventTypeBitmask'] & 0x1) != 0
```

### MC normalisation
The per-event weight that scales a MC sample to the data luminosity:

```
weight = L_total * (sigma * filter_eff * kfactor / sum_of_weights) * mcEventWeights[0] * PileupWeight_NOSYS
```

### Good Run List (GRL)
Select runNumber and lumiBlock, which pass the GRL.

```python
import bisect
import sys
import xml.etree.ElementTree as ET

def parse_grl(path):
    """Parse a GRL XML into {runNumber: sorted list of (firstLB, lastLB)}.
    Ranges are kept as ranges, not expanded into every LB number: a full-year
    GRL is ~24k lumiblocks but only ~270 ranges.
    """
    grl = {}
    for lbc in ET.parse(path).getroot().iter('LumiBlockCollection'):
        run = int(lbc.findtext('Run'))
        ranges = grl.setdefault(run, [])   # merge a run split across blocks
        for lbr in lbc.findall('LBRange'):
            ranges.append((int(lbr.get('Start')), int(lbr.get('End'))))
    for ranges in grl.values():
        ranges.sort()
    return grl

def pass_grl(grl, runnumber, lumiblock):
    """True if (runnumber, lumiblock) is in the GRL. Both ends inclusive."""
    ranges = grl.get(runnumber)
    if ranges is None:
        return False
    i = bisect.bisect_right(ranges, (lumiblock, sys.maxsize)) - 1
    return i >= 0 and ranges[i][1] >= lumiblock

# Data only -- MC has no GRL and must skip this cut entirely.
grl = parse_grl('./your_grl_dir/....xml')

if not pass_grl(grl, runnumber, lumiblock):
    continue
```
The GRL should generally be provided by the user. If it is not provided, an appropriate
GRL may be obtained from CVMFS:
/cvmfs/atlas.cern.ch/repo/sw/database/GroupData/GoodRunsLists/

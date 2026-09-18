# Run2 Open data

## Scope: what "already calibrated" does and does not cover
Object kinematics (`pt`, `eta`, `phi`, `m`) and the ID/isolation decisions in
Open Data are **nominal calibrated values**, so they are read and cut on
directly — no CP tool is needed to reproduce them.

**Scale factors and systematic variations are a separate matter: they are not
stored in these files.** There are no per-object `_NOSYS` branches and no
efficiency, ID, isolation or trigger SF branches; the only pre-computed weight
is `EventInfoAuxDyn.PileupWeight_NOSYS` (MC only, see [events.md](events.md)).
An analysis that needs lepton SFs or an uncertainty band cannot get them from
Open Data alone, and these references do not cover obtaining them.

## Data Metadata
| Year | Integrated Luminosity (pb^{-1})|
| --- | --- |
| 2015 | 3200 |
| 2016 | 32900 |

### Triggers
**The specific triggers used must be selected according to the requirements of the analysis.**
The following are examples of commonly used lowest-threshold triggers, with the
fraction of the recorded luminosity over which each chain ran **unprescaled**.
The available triggers depend on the run/lumi (file). Listing a chain here does not
mean a given file holds its matching branch — test it first, see *Pass trigger* in [triggers.md](triggers.md).

| chain | L1 seed | unprescaled 2015 / 2016 (% of lumi) |
|---|---|---|
| **single muon** | | |
| HLT_mu20_iloose_L1MU15 | L1_MU15 | 98 / 0 |
| HLT_mu24_imedium | L1_MU20 | 98 / 35 |
| HLT_mu26_ivarmedium | L1_MU20 | - / 98 |
| HLT_mu50 | L1_MU20 | 98 / 98 |
| HLT_mu40 | L1_MU20 | 98 / 18 |
| **di/tri muon** | | |
| HLT_2mu10 | L1_2MU10 | 98 / 8 |
| HLT_2mu14 | L1_2MU10 | 98 / 98 |
| HLT_mu18_mu8noL1 | L1_MU15 | 98 / - |
| HLT_mu22_mu8noL1 | L1_MU20 | 98 / 98 |
| HLT_3mu6 | L1_3MU6 | 98 / 98 |
| **single electron** | | |
| HLT_e24_lhmedium_iloose_L1EM20VH | L1_EM20VH | 100 / 0 |
| HLT_e24_lhtight_nod0_iloose | L1_EM20VHI | 100 / 37 |
| HLT_e26_lhtight_iloose | L1_EM22VHI | 100 / 33 |
| HLT_e26_lhtight_nod0_ivarloose | L1_EM22VHI | - / 100 |
| HLT_e60_lhmedium | L1_EM22VHI | 100 / 33 |
| HLT_e60_lhmedium_nod0 | L1_EM22VHI | 100 / 100 |
| HLT_e120_lhloose | L1_EM22VHI | 100 / 8 |
| HLT_e140_lhloose_nod0 | L1_EM22VHI | 100 / 100 |
| **di/tri electron** | | |
| HLT_2e12_lhloose_L12EM10VH | L1_2EM10VH | 100 / - |
| HLT_2e15_lhvloose_L12EM13VH | L1_2EM13VH | 100 / 32 |
| HLT_2e17_lhvloose | L1_2EM15VH | 97 / 33 |
| HLT_2e17_lhvloose_nod0 | L1_2EM15VH | 97 / 100 |
| HLT_e17_lhloose_2e9_lhloose | L1_EM15VH_3EM7 | 100 / 32 |
| HLT_e17_lhloose_nod0_2e9_lhloose_nod0 | L1_EM15VH_3EM7 | 100 / 97 |
| **e-mu** | | |
| HLT_e17_lhloose_mu14 | L1_EM15VH_MU10 | 98 / 31 |
| HLT_e17_lhloose_nod0_mu14 | L1_EM15VH_MU10 | 98 / 98 |
| HLT_e7_lhmedium_mu24 | L1_MU20 | 98 / 31 |
| HLT_e7_lhmedium_nod0_mu24 | L1_MU20 | 98 / 98 |
| HLT_e12_lhloose_2mu10 | L1_2MU10 | 98 / 31 |
| HLT_e12_lhloose_nod0_2mu10 | L1_2MU10 | 98 / 98 |
| HLT_2e12_lhloose_mu10 | L1_2EM8VH_MU10 | 98 / 31 |
| HLT_2e12_lhloose_nod0_mu10 | L1_2EM8VH_MU10 | 98 / 98 |

### Gotchas when subsetting
* **Never scale a subset result by file count**
  Files are typically ordered by run; early files carry far less luminosity.

## Monte Carlo Metadata
One representative sample per major SM process, from the `2024r-pp` release, in
`mc/` — one file per process group (see *Extending these tables*). `mcChannelNumber`
of an event picks the file through the DSID column.

| Process group | File | DSIDs |
|---|---|---|
| V+jets (Sherpa 2.2.11/2.2.14) | [mc/vjets.md](mc/vjets.md) | 700322, 700325, 700794, 700340, 700343, 700337 |
| Top | [mc/top.md](mc/top.md) | 410470, 410471, 410658, 601355 |
| Diboson (Sherpa 2.2.12) | [mc/diboson.md](mc/diboson.md) | 700600, 700601, 700602 |
| ttV | [mc/ttv.md](mc/ttv.md) | 410155, 410218 |
| Higgs (125 GeV) | [mc/higgs.md](mc/higgs.md) | 343981, 345060, 345324, 346214 |
| Photon and QCD | [mc/photon-qcd.md](mc/photon-qcd.md) | 364352, 423103, 364703 |

**A row marked with siblings is one slice of a set, not the whole process.**
Using it alone silently normalises to a fraction of the cross section.

The listed DSID is the one to start from; take the rest of its family from
`get_metadata`. These tables are a quick reference and a fallback for when the
network is unavailable, not the sample list of any analysis — which processes
are needed, and which generator variant, is the analysis's own choice.

### Extending these tables
The table values come from `atlasopenmagic`, which is also how to resolve a
DSID that is not listed here.

```python
import atlasopenmagic as om

om.set_release('2024r-pp')     # pin it; do not rely on the default release
m = om.get_metadata('345060')  # cross_section_pb, genFiltEff, kFactor,
                               # nEvents, sumOfWeights, sumOfWeightsSquared, ...
urls = om.get_urls('345060')   # root://eospublic.cern.ch/... DAOD_PHYSLITE
```

Why these tables are retained instead of being replaced by a function call:
`get_metadata` retrieves metadata over the network on first use.
If outbound network access is unavailable, no metadata can be retrieved,
so the tables in `mc/` serve as the local fallback or quick reference.


### Additional information
* `ID` matches the `mcChannelNumber` in the event information container.
* Do not recalculate the sum of weights from skimmed datasets; use the value in
  these tables, or `get_metadata(dsid)['sumOfWeights']` — they are the same number.

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

| Chain | L1 seed | Signature | 2015 unprescaled | 2016 unprescaled |
|---|---|---|---|---|
| HLT_mu20_iloose_L1MU15 | L1_MU15 | single muon | 97.98% | 0.06% |
| HLT_mu24_imedium | L1_MU20 | single muon | 98.36% | 35.34% |
| HLT_mu26_ivarmedium | L1_MU20 | single muon | - | 98.43% |
| HLT_mu50 | L1_MU20 | single muon | 98.36% | 98.32% |
| HLT_mu40 | L1_MU20 | single muon | 98.36% | 17.55% |
| HLT_2mu10 | L1_2MU10 | di/tri muon | 97.98% | 7.62% |
| HLT_2mu14 | L1_2MU10 | di/tri muon | 97.98% | 98.43% |
| HLT_mu18_mu8noL1 | L1_MU15 | di/tri muon | 97.98% | - |
| HLT_mu22_mu8noL1 | L1_MU20 | di/tri muon | 98.36% | 98.32% |
| HLT_3mu6 | L1_3MU6 | di/tri muon | 97.98% | 98.43% |
| HLT_e24_lhmedium_iloose_L1EM20VH | L1_EM20VH | single electron | 99.99% | 0.14% |
| HLT_e24_lhtight_nod0_iloose | L1_EM20VHI | single electron | 99.99% | 36.76% |
| HLT_e26_lhtight_iloose | L1_EM22VHI | single electron | 99.99% | 32.70% |
| HLT_e26_lhtight_nod0_ivarloose | L1_EM22VHI | single electron | - | 99.81% |
| HLT_e60_lhmedium | L1_EM22VHI | single electron | 99.99% | 32.70% |
| HLT_e60_lhmedium_nod0 | L1_EM22VHI | single electron | 99.99% | 99.81% |
| HLT_e120_lhloose | L1_EM22VHI | single electron | 99.99% | 7.84% |
| HLT_e140_lhloose_nod0 | L1_EM22VHI | single electron | 99.83% | 99.81% |
| HLT_2e12_lhloose_L12EM10VH | L1_2EM10VH | di/tri electron | 99.99% | - |
| HLT_2e15_lhvloose_L12EM13VH | L1_2EM13VH | di/tri electron | 99.83% | 32.38% |
| HLT_2e17_lhvloose | L1_2EM15VH | di/tri electron | 97.35% | 32.70% |
| HLT_2e17_lhvloose_nod0 | L1_2EM15VH | di/tri electron | 97.35% | 99.93% |
| HLT_e17_lhloose_2e9_lhloose | L1_EM15VH_3EM7 | di/tri electron | 99.83% | 32.38% |
| HLT_e17_lhloose_nod0_2e9_lhloose_nod0 | L1_EM15VH_3EM7 | di/tri electron | 99.83% | 96.98% |
| HLT_e17_lhloose_mu14 | L1_EM15VH_MU10 | e-mu | 97.98% | 31.33% |
| HLT_e17_lhloose_nod0_mu14 | L1_EM15VH_MU10 | e-mu | 97.98% | 98.37% |
| HLT_e7_lhmedium_mu24 | L1_MU20 | e-mu | 98.36% | 31.21% |
| HLT_e7_lhmedium_nod0_mu24 | L1_MU20 | e-mu | 98.36% | 98.43% |
| HLT_e12_lhloose_2mu10 | L1_2MU10 | e-mu | 97.98% | 31.33% |
| HLT_e12_lhloose_nod0_2mu10 | L1_2MU10 | e-mu | 97.98% | 98.43% |
| HLT_2e12_lhloose_mu10 | L1_2EM8VH_MU10 | e-mu | 97.98% | 31.33% |
| HLT_2e12_lhloose_nod0_mu10 | L1_2EM8VH_MU10 | e-mu | 97.98% | 98.37% |

## Monte Carlo Metadata
One representative sample per major SM process, from the `2024r-pp` release (see
*Extending this table*).

| ID | Physics short | Process | Cross section (pb) | Filter efficiency | K-factor | Sum of weights | Sum of weights squared | Siblings |
|---|---|---|---|---|---|---|---|---|
| **V+jets (Sherpa 2.2.11/2.2.14) — one of a 3-slice flavour family, see below** |||||||||
| 700322 | Sh_2211_Zee_maxHTpTV2_CVetoBVeto | Z->ee without b- or c-quarks | 2221.2 | 0.8460721 | 1.0 | 1195987492151594.8 | 9.638225966800312e+22 | 700320-700322 |
| 700325 | Sh_2211_Zmumu_maxHTpTV2_CVetoBVeto | Z->mumu without b- or c-quarks | 2221.3 | 0.8463929 | 1.0 | 1196323720518196.5 | 9.622751117378584e+22 | 700323-700325 |
| 700794 | Sh_2214_Ztautau_maxHTpTV2_CVetoBVeto | Z->tautau without b- or c-quarks | 2239.6 | 0.845961 | 1.0 | 721456451073821.5 | 5.896472607444247e+22 | 700792-700794 |
| 700340 | Sh_2211_Wenu_maxHTpTV2_CVetoBVeto | W->enu | 21742.0 | 0.8435958 | 1.0 | 2.897842369250881e+16 | 2.463680354054539e+25 | 700338-700340 |
| 700343 | Sh_2211_Wmunu_maxHTpTV2_CVetoBVeto | W->munu | 21806.0 | 0.8435538 | 1.0 | 2.5735614820769316e+16 | 2.1940065983114303e+25 | 700341-700343 |
| 700337 | Sh_2211_Znunu_pTV2_CVetoBVeto | Z->nunu | 447.13 | 0.712717 | 1.0 | 141602625733007.44 | 4.05027168968272e+21 | 700335-700337 |
| **Top** |||||||||
| 410470 | PhPy8EG_A14_ttbar_hdamp258p75_nonallhad | ttbar | 729.77 | 0.5437965 | 1.139756362 | 24014984445.816772 | 17816963736313.004 | — |
| 410471 | PhPy8EG_A14_ttbar_hdamp258p75_allhad | ttbar | 729.77 | 0.4562069 | 1.139740744 | 12684451756.43042 | 9410150225002.22 | — |
| 410658 | PhPy8EG_A14_tchan_BW50_lept_top | t-channel single top | 36.996 | 1.0 | 1.1935 | 124412662.10998063 | 4978964942.567033 | 410659 = antitop |
| 601355 | PhPy8EG_tW_dyn_DR_incl_top | tW single top | 36.003 | 1.0 | 1.0 | 95054189.43354416 | 3463158468.00062 | 601352 = antitop |
| **Diboson (Sherpa 2.2.12)** |||||||||
| 700600 | Sh_2212_llll | VV->llll | 1.2974 | 1.0 | 1.0 | 8900161062.01826 | 732258484286478.2 | — |
| 700601 | Sh_2212_lllv | VV->lllnu | 4.661 | 1.0 | 1.0 | 73236556201.67273 | 3.586932793917845e+16 | — |
| 700602 | Sh_2212_llvv_os | VV->llnunu | 12.079 | 1.0 | 1.0 | 213218033453.56433 | 1.51167341602775e+17 | — |
| **ttV** |||||||||
| 410155 | aMcAtNloPythia8EvtGen_MEN30NLO_A14N23LO_ttW | ttW | 0.54822 | 1.0 | 1.096 | 54486.1498632431 | 74269.99889367796 | — |
| 410218 | aMcAtNloPythia8EvtGen_MEN30NLO_A14N23LO_ttee | tt+ee | 0.036864 | 1.0 | 1.12 | 2585.2198036387563 | 421.94472719772057 | 410219 = tt+mumu |
| **Higgs (125 GeV)** |||||||||
| 343981 | PowhegPythia8EvtGen_NNLOPS_nnlo_30_ggH125_gamgam | H->gamma gamma GGF production | 28.3 | 0.00227 | 1.717 | 5660114.774513245 | 162287375.54376578 | — |
| 345060 | PowhegPythia8EvtGen_NNLOPS_nnlo_30_ggH125_ZZ4l | ggH H->ZZ->llll | 28.3 | 0.000124 | 1.717 | 45231011.19517517 | 1296676130.5944173 | — |
| 345324 | PowhegPythia8EvtGen_NNLOPS_NN30_ggH125_WWlvlv_EF_15_5 | gg->H->WW*->lvlv | 28.3 | 0.02338 | 1.717 | 4726922.222862244 | 135509958.57900387 | — |
| 346214 | PowhegPy8EG_NNPDF30_AZNLOCTEQ6L1_VBFH125_gamgam | VBF H->gamma gamma | 3.782 | 0.00227 | 1.0 | 299915.6851153374 | 1142827.2423424923 | — |
| **Photon and QCD — one slice of a family, see below** |||||||||
| 364352 | Sherpa_224_NNPDF30NNLO_Diphoton_myy_90_175 | Diphoton | 51.822 | 1.0 | 1.0 | 3083770.984963063 | 5585853.27312259 | 364350-364354 |
| 423103 | Pythia8EvtGen_A14NNPDF23LO_gammajet_DP70_140 | QCD direct photon production | 106210000.0 | 3.93e-05 | 1.0 | 1989000.0 | 1989000.0 | 423099-423112 |
| 364703 | Pythia8EvtGen_A14NNPDF23LO_jetjet_JZ3WithSW | QCD jets | 26450000.0 | 0.01165833 | 1.0 | 364.7516766097338 | 0.008413246890465525 | 364700-364712 |

**A row marked with siblings is one slice of a set, not the whole process.**
Using it alone silently normalises to a fraction of the cross section.

The listed DSID is the one to start from; take the rest of its family from
`get_metadata`. This table is a quick reference and a fallback for when the
network is unavailable, not the sample list of any analysis — which processes
are needed, and which generator variant, is the analysis's own choice.

### Extending this table
The table values come from `atlasopenmagic`, which is also how to resolve a
DSID that is not listed here.

```python
import atlasopenmagic as om

om.set_release('2024r-pp')     # pin it; do not rely on the default release
m = om.get_metadata('345060')  # cross_section_pb, genFiltEff, kFactor,
                               # nEvents, sumOfWeights, sumOfWeightsSquared, ...
urls = om.get_urls('345060')   # root://eospublic.cern.ch/... DAOD_PHYSLITE
```

Why this table is retained instead of being replaced by a function call:
`get_metadata` retrieves metadata over the network on first use.
If outbound network access is unavailable, no metadata can be retrieved,
so the table above serves as the local fallback or quick reference.


### Additional information
* `ID` matches the `mcChannelNumber` in the event information container.
* Do not recalculate the sum of weights from skimmed datasets; use the value in
  this table, or `get_metadata(dsid)['sumOfWeights']` — they are the same number.

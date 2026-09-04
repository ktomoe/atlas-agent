# Analysis triggers

## List of containers
| Name | Purpose | Prefix |
|---|---|---|
| AnalysisTrigMatch_HLT_{chain} | Offline objects matched to the HLT chain `{chain}`. One container per chain. | AuxDyn |

### Additional information
* There are hundreds of AnalysisTrigMatch_HLT* containers. To minimize token usage, narrow them down with `filter_name`.

### Trigger naming convention
{multiplicity|none}{e|mu|g}{threshold in GeV}{_qualifier}...{_L1seed|none}

* `e120_lhloose` -> single electron, 120 GeV threshold, likelihood Loose ID
* `2mu10` -> two muons, 10 GeV threshold each
* `mu24_imedium` -> single muon, 24 GeV, medium isolation
* `e24_lhmedium_iloose_L1EM20VH` -> single electron, 24 GeV, likelihood Medium ID,
  loose isolation, seeded by the L1 item `EM20VH`
* `g35_loose_g25_loose` -> two photons with different thresholds, spelled out separately

Qualifiers are ID (`lhloose`, `medium`), isolation (`iloose`, `imedium`) or
reconstruction variants (`nod0`, `msonly`) — not algorithm names.

## AnalysisTrigMatch_HLT_{chain} branches
| Name | Type | Definition | Unit |
|---|---|---|---|
| TrigMatchedObjects | vector<vector<ElementLink<DataVector<xAOD::IParticle>>>> | Links to the **offline** objects that matched this chain. Outer index: one entry per matched object. Inner index: the links that object holds. ||

### Additional Information
* `xTrigDecision` (m_persKey=148942056) itself is not readable with uproot, but its Aux branches are:
  `xTrigDecisionAux./xTrigDecisionAux.{tbp,tap,tav}` hold the L1 decision words
  (before prescale / after prescale / after veto) and read as `var * uint32`.
* The raw **HLT** decision per chain is in the same container
  (`efPassedPhysics`, `efPassedRaw`, `efPrescaled`; Run 2 reuses the EF fields).
  It is **not** used in these references because the procedure is complex (see `./trigger-advanced.md`),
  so use the trigger matching above unless the raw decision is really needed.

## Examples of pseudo code
### Pass trigger
Only matched objects are stored, so a non-empty entry means the event fired the
chain **and** an offline object matched it. This is a matching requirement, not
the raw decision.

```python
branch = 'AnalysisTrigMatch_HLT_e24_lhmedium_L1EM20VHAuxDyn.TrigMatchedObjects'
if branch not in tree:      # no match anywhere in this file -> every event fails
    return False

matched = event[branch]
if len(matched) == 0:
    return False
```

### Which offline object matched
The type is `vector<vector<ElementLink<...>>>`, i.e. the **not split** layout in
io.md: index by object, then by link, then by field.

```python
link = event['AnalysisTrigMatch_HLT_e24_lhmedium_L1EM20VHAuxDyn.TrigMatchedObjects'][ii][0]

if link['m_persKey'] == 0:              # unset link
    continue

offline_index = link['m_persIndex']     # index into the container m_persKey names
```

The links are to `xAOD::IParticle`, so **the target container is not fixed by the
type**: an e-chain points into `AnalysisElectrons`, a mu-chain into
`AnalysisMuons`. Resolve it from `m_persKey` instead of assuming one.

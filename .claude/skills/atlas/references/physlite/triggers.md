# Analysis triggers

## List of containers
| Name | Purpose | Prefix |
|---|---|---|
| AnalysisTrigMatch_HLT_{chain} | Offline objects matched to the HLT chain `{chain}`. One container per chain. | AuxDyn |

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
  It is **not** used in these references because the procedure is complex (see `Raw HLT decision` below),
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

## Raw HLT decision
The decision word is in `CollectionTree`, but the chain -> bit mapping that
decodes it is in `MetaData`, next to the prescale payloads.

| What | Branch |
|---|---|
| HLT decision words | `CollectionTree` / `xTrigDecisionAux./xTrigDecisionAux.{efPassedPhysics,efPassedRaw,efPrescaled}` |
| Chain `counter`, i.e. the bit index | `MetaData` / `TriggerMenuJson_HLTAux./TriggerMenuJson_HLTAux.{key,payload}` |
| Menu applying to the event | `CollectionTree` / `TrigConfKeys` -> `m_smk` |

Each decision branch reads as `var * uint32`: 256 words per event, i.e. one
8192-bit set indexed by the chain's `counter`. `efPassedPhysics` is the decision
after prescale and veto -- the one an analysis cuts on. `efPassedRaw` is before
prescale, and `efPrescaled` marks the chain as prescaled out in that event.

**`counter` is a property of the menu, not of the chain name.** It is looked up
per `m_smk`, and the same caveat as the prescale tables applies: build the
{smk: counter} map from **every** file that will be read, because a file's menu
payload holds only the menus written into that file.

```python
def hlt_counters(paths, chain):
    """{m_smk: bit index of `chain`} over every file that will be read.
    A smk is absent from the map when its menu does not hold the chain.
    """
    mb = "TriggerMenuJson_HLTAux./TriggerMenuJson_HLTAux."
    counters = {}
    for path in paths:
        md = uproot.open(path)["MetaData"]
        for key, payload in zip(md[mb + "key"].array(library="np")[0],
                                md[mb + "payload"].array(library="np")[0]):
            entry = json.loads(payload)["chains"].get(chain)
            if entry is not None:
                counters[int(key)] = int(entry["counter"])
    return counters

counters = hlt_counters(files, 'HLT_e24_lhmedium_L1EM20VH')

smk = event['TrigConfKeys']['m_smk']

# An smk with no counter means the chain is not in that menu at all -- the
# event cannot have fired it. This is a real failed cut, unlike an unresolved
# prescale key, which is a bug in the file list.
if smk not in counters:
    continue

bit = counters[smk]
word = event['xTrigDecisionAux./xTrigDecisionAux.efPassedPhysics'][bit // 32]

if not (word >> (bit % 32)) & 0x1:
    continue
```

## Prescales
**DATA Only** Prescale values are **not** in `CollectionTree`. They live in the `MetaData`
tree as JSON payloads; `CollectionTree` only carries the key that selects which
set applies to the event.

| What | Branch |
|---|---|
| HLT prescale values | `MetaData` / `TriggerMenuJson_HLTPSAux./TriggerMenuJson_HLTPSAux.{key,payload}` |
| L1 prescale values | `MetaData` / `TriggerMenuJson_L1PSAux./TriggerMenuJson_L1PSAux.{key,payload}` |
| Trigger menu (L1 seed of a chain) | `MetaData` / `TriggerMenuJson_HLTAux./TriggerMenuJson_HLTAux.{key,payload}` |
| Key applying to the event | `CollectionTree` / `TrigConfKeys` -> `m_smk`, `m_l1psk`, `m_hltpsk` |

`TrigConfKeys` reads as a record of three uint32, one per event.
The total prescale is **L1 x HLT**, so both sides must be looked up.

```python
def parse_prescales(paths, chain, l1item):
    """{m_hltpsk: HLT prescale}, {m_l1psk: L1 prescale} for one chain.
    -1.0 where the chain / L1 item is disabled or absent from that set.

    `paths` must list **every** file that will be read. The key branch of a
    file holds only the keys written into that file, and the key sets of
    different files do not overlap, so a table built from one file resolves
    the keys of essentially no other file.

    The two payloads have different shapes and cannot share a parser: HLT
    keeps a `prescales` map with the value in `prescale` (a string), L1 keeps
    a `cutValues` map with no prescale field -- it is recovered from `cut` as
    0xFFFFFF / (0x1000000 - cut), which reproduces the value quoted in the
    item's `info` field exactly. `enabled` is the string "true"/"false" in
    both.
    """
    hb = "TriggerMenuJson_HLTPSAux./TriggerMenuJson_HLTPSAux."
    lb = "TriggerMenuJson_L1PSAux./TriggerMenuJson_L1PSAux."
    hlt, l1 = {}, {}
    for path in paths:
        md = uproot.open(path)["MetaData"]
        for key, payload in zip(md[hb + "key"].array(library="np")[0],
                                md[hb + "payload"].array(library="np")[0]):
            e = json.loads(payload)["prescales"].get(chain)
            hlt[int(key)] = float(e["prescale"]) if e and e["enabled"] == "true" else -1.0
        for key, payload in zip(md[lb + "key"].array(library="np")[0],
                                md[lb + "payload"].array(library="np")[0]):
            e = json.loads(payload)["cutValues"].get(l1item)
            l1[int(key)] = (0xFFFFFF / (0x1000000 - int(e["cut"]))
                            if e and e["enabled"] == "true" else -1.0)
    return hlt, l1

# Data only -- MC carries TrigConfKeys but no prescale metadata at all.
hlt_ps, l1_ps = parse_prescales(files, 'HLT_e24_lhmedium_L1EM20VH', 'L1_EM20VH')

keys = event['TrigConfKeys']
hltpsk, l1psk = keys['m_hltpsk'], keys['m_l1psk']

# An unresolved key is a bug in the file list, not a disabled chain. Raise on
# it; treating it as a failed cut would drop those events without a word.
if hltpsk not in hlt_ps or l1psk not in l1_ps:
    raise KeyError(f"prescale keys {hltpsk}/{l1psk} not in the table: "
                   "parse the metadata of every file being read")

if hlt_ps[hltpsk] < 0 or l1_ps[l1psk] < 0:
    continue        # chain disabled in this event's prescale set

# The total prescale is L1 x HLT.
prescale = l1_ps[l1psk] * hlt_ps[hltpsk]
```

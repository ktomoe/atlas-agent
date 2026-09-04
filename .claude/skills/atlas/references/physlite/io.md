# Reading PHYSLITE files

Variables are stored in an auxiliary store, so the on-disk branch name is the
container name with an `Aux` or `AuxDyn` suffix, e.g. the pT of `AnalysisMuons`
is the branch `AnalysisMuonsAuxDyn.pt`. The variables for physics objects are
stored in AuxDyn.

## Python (uproot)
Two patterns, chosen by how much data is read. Common setup for both:

```python
import glob
import uproot
import awkward as ak
import numpy as np

files = {f: "CollectionTree" for f in sorted(glob.glob("./your_data_dir/*"))}
branches = [
    "AnalysisMuonsAuxDyn.pt",
    "AnalysisMuonsAuxDyn.eta",
    "AnalysisMuonsAuxDyn.quality",
]
```
### Check the branch names
The per-container references list the branches an analysis normally needs, not
everything a file holds. Resolve any other name against the file itself.
A PHYSLITE `CollectionTree` holds ~700 branches and a single container can hold 90.
To minimize token usage, narrow them down with `filter_name`.

```python
tree = uproot.open(next(iter(files)))["CollectionTree"]
"AnalysisMuonsAuxDyn.quality" in tree                    # prefer this
tree.keys(filter_name="AnalysisElectronsAuxDyn.*cone*")  # then this
tree.keys(filter_name="AnalysisMuonsAuxDyn.*")           # only if you must
```
Test existence with `name in tree`, not with `name in tree.keys()`: for a
split branch the two disagree.

#### Dynamic branches are per file
AuxDyn branches are written only when the producer filled them, so the branch
set differs from file to file. Trigger matching is the most common case, but
any low-multiplicity container may be absent.

### Count the total events
```python
total = sum(n for _, _, n in uproot.num_entries(files))
```

### Small inputs: load once with `uproot.concatenate`
Use when the requested branches fit comfortably in memory. Everything is one
array, so the selection is written once and the result can be re-binned or
re-cut without re-reading the files.

```python
array = uproot.concatenate(files, branches)

# A cut is one masking expression over the whole jagged array -- no event loop.
tight_pt = array["AnalysisMuonsAuxDyn.pt"][(array["AnalysisMuonsAuxDyn.quality"] & 0x3) == 0]
```

### Large inputs: stream with `uproot.iterate`
Use when they do not. Memory stays at one chunk, so results must be
**accumulated** across chunks (histograms, sums, counters) — never by
concatenating the chunks back together. `step_size` is a memory budget, not an
event count.

```python
for chunk in uproot.iterate(files, branches, step_size="100 MB"):
    # do chunk processing

    # or, event by event processing
    for event in chunk:
        # do something
```
See ./vectorized-recipes.md for chunk processing.

### Parallel processing
File-level parallelism is also appropriate, for example using `multiprocessing.Pool`. 
Files are independent, may contain different sets of AuxDyn branches,
and can be processed in parallel for a several-fold speedup.

## ElementLink branches
The on-disk layout depends on how many links each object holds,
and the two layouts are read with **different syntax**.
The types below are written exactly as they appear in the Type column of the
per-container references, so they can be matched literally.

| Type in the container reference | Links per object | Layout and how to read |
|---|---|---|
| `vector<ElementLink<...>>` | one | **Split.** ROOT stores it as two sub-branches. Read `<branch>.m_persKey` and `<branch>.m_persIndex` as separate branches, then index by object: `[ii]` |
| `vector<vector<ElementLink<...>>>` | many | **Not split.** Read `<branch>` itself; it is a record array. Index by object, then by link, then by field: `[ii][jj]['m_persIndex']` |

**`m_persKey == 0` means the link is not set.** Always check it before using
`m_persIndex`, otherwise index 0 of the target container is read by mistake.

### Resolving `m_persKey` to a container
`m_persKey` is a hash of the **container name**, not an index into anything.
The target container is therefore fixed by the key, not by the link type: an
`ElementLink<xAOD::IParticle>` reaches whichever container the producer wrote.
Resolve the key; never infer the container from the branch it was read from.

These keys are a hash of the name alone, so they are the **same in data and MC**
and across streams (`DAOD_PHYSLITE`, skimmed `D2AOD_*`).

## Athena / AnalysisBase
**Athena / AnalysisBase are not supported in these PHYSLITE references.**

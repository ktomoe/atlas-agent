# Vectorized examples (awkward)
Each example shows one *mechanic*; cuts that reuse a shown mechanic are
deliberately absent. Cut values are illustrative, not ATLAS working points.

## Pitfalls the code below relies on
- **P1** A mask must be `var * bool`, never option type: `ak.sum`/`ak.any`
  skip `None` instead of counting it as a failure, so cut flows stop adding up.
  `ak.fill_none` every `None` away before it reaches a mask.
- **P2** An event-level `None` (e.g. no PV) inside jagged arithmetic turns the
  whole event's object list into `None`, and `fill_none(axis=-1)` does not undo
  it. Keep it as a separate event-level bool and `&` it in — it broadcasts.
- **P3** Before gathering through a link require `m_persKey == target` **and**
  `m_persIndex < ak.num(target)` (thinned containers leave dangling indices,
  and an out-of-range jagged index raises). Replace failing indices with 0 and
  keep the `linked` mask to drop those objects afterwards.
- **P4** Pad the target container to length >= 1 before indexing: index 0 is
  itself out of range in an event with no track. The padded entry is only read
  where `linked` is False.
- **P5** Link 0 of a not-split link: `ak.firsts(links, axis=2)`, not
  `links[:, :, 0]` — an empty link list makes the slice raise.
- **P6** The right index read from the wrong container returns a number and
  raises nothing. Resolve the container from the key (io.md).
- **P7** Per-element clamp with `np.maximum`, never `max()`.
- **P8** `ak.cartesian(..., nested=True)` keeps the outer axis aligned with the
  first argument so `axis=2` reduces over the second; an empty inner list gives
  `False` from `ak.any`, so objects with nothing to compare need no special case.

## Invariant mass
Muons carry no mass branch (../parameters.md); electrons have `m`.
```python
MU_MASS = 105.6583755   # MeV

pairs = ak.combinations(ak.zip({"pt": pt, "eta": eta, "phi": phi, "q": charge}),
                        2, axis=1, fields=["l0", "l1"])
pairs = pairs[pairs.l0.q * pairs.l1.q < 0]        # opposite sign

def energy(o, mass):
    return np.sqrt((o.pt*np.cosh(o.eta))**2 + mass**2)

E  = energy(pairs.l0, MU_MASS) + energy(pairs.l1, MU_MASS)
px = pairs.l0.pt*np.cos(pairs.l0.phi)  + pairs.l1.pt*np.cos(pairs.l1.phi)
py = pairs.l0.pt*np.sin(pairs.l0.phi)  + pairs.l1.pt*np.sin(pairs.l1.phi)
pz = pairs.l0.pt*np.sinh(pairs.l0.eta) + pairs.l1.pt*np.sinh(pairs.l1.eta)
mass = np.sqrt(np.maximum(E**2 - (px**2 + py**2 + pz**2), 0.0))   # MeV (P7)
```

## Gather through a link: |z0 sin θ| < 0.5 mm
Shown for muons (split link → `InDetTrackParticles`); the electron variant
(not-split link → `GSFTrackParticles`) differs only in the four lines at the end.
```python
hard_pv = array["PrimaryVerticesAuxDyn.z"][array["PrimaryVerticesAuxDyn.vertexType"] == 1]
has_pv = ak.num(hard_pv) > 0                       # event-level bool (P2)
pvz = ak.fill_none(ak.firsts(hard_pv), 0.0)        # value irrelevant where ~has_pv

key = array["AnalysisMuonsAuxDyn.inDetTrackParticleLink.m_persKey"]
idx = array["AnalysisMuonsAuxDyn.inDetTrackParticleLink.m_persIndex"]
TRK = "InDetTrackParticlesAuxDyn."                 # 490246363, tracks.md (P6)

linked = (key == 490246363) & (idx < ak.num(array[TRK + "z0"]))   # P3
idx = ak.where(linked, idx, 0)

def by_track(name):                                                 # P4
    return ak.fill_none(ak.pad_none(array[TRK + name], 1), 0.0)[idx]

z0sin = ((by_track("z0") + by_track("vz")) - pvz) * np.sin(by_track("theta"))
mu_pass = has_pv & linked & (abs(z0sin) < 0.5)                     # var * bool (P1)
```
Electron variant — replace the `key`/`idx`/`TRK` lines and the key value:
```python
first = ak.firsts(array["AnalysisElectronsAuxDyn.trackParticleLinks"], axis=2)  # P5
key = ak.fill_none(first.m_persKey, 0)     # unset link -> 0, fails the key test (P1)
idx = ak.fill_none(first.m_persIndex, 0)
TRK = "GSFTrackParticlesAuxDyn."           # key 776133387
```

## ΔR and overlap removal
Defined on objects that already passed their own selection.
```python
def delta_r(a, b):
    dphi = (a.phi - b.phi + np.pi) % (2*np.pi) - np.pi   # wrap required, ../coordinates.md
    return np.sqrt((a.eta - b.eta)**2 + dphi**2)

el = ak.zip({"eta": array["AnalysisElectronsAuxDyn.eta"], "phi": array["AnalysisElectronsAuxDyn.phi"]})[el_pass]
mu = ak.zip({"eta": array["AnalysisMuonsAuxDyn.eta"],     "phi": array["AnalysisMuonsAuxDyn.phi"]})[mu_pass]

pairs = ak.cartesian({"el": el, "mu": mu}, axis=1, nested=True)     # P8
el = el[~ak.any(delta_r(pairs.el, pairs.mu) < 0.2, axis=2)]         # drop electrons near a muon
```

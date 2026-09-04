# Vectorized examples (awkward)
**The examples are not exhaustive, and are not meant to be.**
Each one demonstrates a *mechanic* — a link layout, a broadcast, an option-type trap —
never a catalogue of cuts. A cut that reuses a mechanic already shown is
**deliberately absent**. Choosing the remaining cuts and optimising them is
the work of the analysis and of the agent reading this; it is not a gap here.

## Invariant mass
Vectorized counterpart of [kinematics.md](kinematics.md). Muons carry no mass
branch, so the mass comes from `../parameters.md`; electrons have `m`.

```python
MU_MASS = 105.6583755   # MeV, ../parameters.md

pairs = ak.combinations(ak.zip({"pt": pt, "eta": eta, "phi": phi, "q": charge}),
                        2, axis=1, fields=["l0", "l1"])
pairs = pairs[pairs.l0.q * pairs.l1.q < 0]        # opposite sign

def energy(o, mass):
    return np.sqrt((o.pt*np.cosh(o.eta))**2 + mass**2)   # p = pt*cosh(eta)

E  = energy(pairs.l0, MU_MASS) + energy(pairs.l1, MU_MASS)
px = pairs.l0.pt*np.cos(pairs.l0.phi)  + pairs.l1.pt*np.cos(pairs.l1.phi)
py = pairs.l0.pt*np.sin(pairs.l0.phi)  + pairs.l1.pt*np.sin(pairs.l1.phi)
pz = pairs.l0.pt*np.sinh(pairs.l0.eta) + pairs.l1.pt*np.sinh(pairs.l1.eta)

# np.maximum, not max(): the clamp is per element of a jagged array.
mass = np.sqrt(np.maximum(E**2 - (px**2 + py**2 + pz**2), 0.0))   # MeV
```

## Muon z0 selection
**The code below is an example, not an official ATLAS working point**
Vectorized counterpart of the `|z0 sin(θ)| < 0.5 mm` cut in
[tracks.md](tracks.md). The muon carries no z0 of its own, so the cut is a
gather from `InDetTrackParticles` through `inDetTrackParticleLink`.

```python
# 1. Per-event primary vertex z: one scalar per event, so it broadcasts
#    against the jagged muon arrays with no further work.
#    ak.firsts gives None where the event has no PV. Do not carry that None
#    into the arithmetic: an event-level None makes the whole muon list of that
#    event None, and ak.fill_none(..., axis=-1) does NOT flatten it back to
#    per-muon False. Split it into an explicit event-level requirement instead.
pv_z = array["PrimaryVerticesAuxDyn.z"]
pv_type = array["PrimaryVerticesAuxDyn.vertexType"]
hard_pv = pv_z[pv_type == 1]
has_pv = ak.num(hard_pv) > 0                 # event-level, plain bool
pvz = ak.fill_none(ak.firsts(hard_pv), 0.0)  # value is irrelevant where ~has_pv

# 2. Sanitize the link BEFORE gathering. m_persIndex is meaningless where the
#    link is unset, so it is replaced by 0 and `linked` drops those muons again
#    in step 4. Range-check the index against the track count of the *same*
#    event as well: a thinned or absent target container leaves stored indices
#    pointing past the end, and an out-of-range jagged index raises.
key = array["AnalysisMuonsAuxDyn.inDetTrackParticleLink.m_persKey"]
idx = array["AnalysisMuonsAuxDyn.inDetTrackParticleLink.m_persIndex"]
n_trk = ak.num(array["InDetTrackParticlesAuxDyn.z0"])   # event-level, broadcasts
linked = (key == 490246363) & (idx < n_trk)  # InDetTrackParticles, see tracks.md
idx = ak.where(linked, idx, 0)

# 3. Gather: indexing a jagged array with a jagged index array is done per
#    event, and the result is aligned with the muons, not with the tracks.
#    Index 0 is itself out of range in an event that holds no track at all --
#    rare but real, so pad every track list to length >= 1 first. The padded
#    entry is only ever read where `linked` is False, and step 4 drops it.
def by_track(name):
    return ak.fill_none(ak.pad_none(array[name], 1), 0.0)[idx]

z0 = by_track("InDetTrackParticlesAuxDyn.z0")
vz = by_track("InDetTrackParticlesAuxDyn.vz")
theta = by_track("InDetTrackParticlesAuxDyn.theta")

# 4. The cut. `has_pv` broadcasts from one value per event to every muon of
#    that event, so an event with no PV contributes no muon.
z0sin = ((z0 + vz) - pvz) * np.sin(theta)
mu_pass = has_pv & linked & (abs(z0sin) < 0.5)
```

`mu_pass` is a jagged boolean aligned with `AnalysisMuons`, of type
`var * bool` and **not** `option[var * bool]`. Combine it with the other muon
requirements by `&`, and apply the result once at the end. Keeping option types
out of the mask matters: `None` entries are skipped by `ak.sum` and `ak.any`
rather than counted as failures, so a cut flow built on a masked array reports
numbers that do not add up.

## Electron z0 selection -- gathering through a not-split link
**The code below is an example, not an official ATLAS working point**
The same `|z0 sin(θ)| < 0.5 mm` cut, for electrons. The physics is identical to
the muon case above; what changes is the link layout. `trackParticleLinks` is
`vector<vector<ElementLink<...>>>`, the **not split** case in [io.md](io.md), so
there are no `.m_persKey` / `.m_persIndex` sub-branches to read: the key and the
index are fields of a record one level *deeper* than the electron.

```python
# 1. Primary vertex z, exactly as in the muon example above.
pv_z = array["PrimaryVerticesAuxDyn.z"]
pv_type = array["PrimaryVerticesAuxDyn.vertexType"]
hard_pv = pv_z[pv_type == 1]
has_pv = ak.num(hard_pv) > 0
pvz = ak.fill_none(ak.firsts(hard_pv), 0.0)

# 2. The primary GSF track is link 0 of the electron. Take it with
#    ak.firsts(..., axis=2) and NOT with links[:, :, 0]: an electron whose link
#    list is empty makes the plain slice raise IndexError, while ak.firsts
#    yields None there and step 3 turns that into a failing electron.
links = array["AnalysisElectronsAuxDyn.trackParticleLinks"]  # var * var * link
first = ak.firsts(links, axis=2)                             # var * ?link

# 3. Fields are read straight through the option, then the None is filled away
#    at once so the mask never becomes an option type (see the note above).
#    An unset link arrives as key == 0, which fails the same comparison.
key = ak.fill_none(first.m_persKey, 0)
idx = ak.fill_none(first.m_persIndex, 0)

n_trk = ak.num(array["GSFTrackParticlesAuxDyn.z0"])   # event-level, broadcasts
linked = (key == 776133387) & (idx < n_trk)  # GSFTrackParticles, see tracks.md
idx = ak.where(linked, idx, 0)

# 4. Gather, padded exactly as in the muon example above. Electrons link to
#    GSFTrackParticles, not to InDetTrackParticles: the same index read from
#    the wrong container is a valid index into an unrelated track, so it
#    returns a number and raises nothing.
def by_track(name):
    return ak.fill_none(ak.pad_none(array[name], 1), 0.0)[idx]

z0 = by_track("GSFTrackParticlesAuxDyn.z0")
vz = by_track("GSFTrackParticlesAuxDyn.vz")
theta = by_track("GSFTrackParticlesAuxDyn.theta")

z0sin = ((z0 + vz) - pvz) * np.sin(theta)
el_pass = has_pv & linked & (abs(z0sin) < 0.5)
```

## ΔR and overlap removal
**The code below is an example, not an official ATLAS working point**
Vectorized counterpart of kinematics.md. Overlap removal is
defined on objects that already passed their own selection, so the masks from
the sections above are applied first: every array here is aligned with the
selected electrons, not with the full container.

```python
def delta_r(a, b):
    """a, b carry eta/phi. The wrap is required -- see ../coordinates.md."""
    dphi = (a.phi - b.phi + np.pi) % (2*np.pi) - np.pi
    return np.sqrt((a.eta - b.eta)**2 + dphi**2)

el = ak.zip({"eta": array["AnalysisElectronsAuxDyn.eta"],
          "phi": array["AnalysisElectronsAuxDyn.phi"]})[el_pass]
mu = ak.zip({"eta": array["AnalysisMuonsAuxDyn.eta"],
          "phi": array["AnalysisMuonsAuxDyn.phi"]})[mu_pass]

# nested=True groups the pairs by electron -- var * var * {el, mu} -- so the
# outer axis stays aligned with el and axis=2 reduces over the muons.
# Without it the pairs are one flat list per event and that alignment is lost.
pairs = ak.cartesian({"el": el, "mu": mu}, axis=1, nested=True)
dr = delta_r(pairs.el, pairs.mu)

# An electron with no muon to check against gets an empty inner list, and
# ak.any over an empty list is False -- the electron is kept and the mask stays
# var * bool, never an option type (see the note above). Events with no muon
# at all behave the same way, so they need no special case.
el_keep = ~ak.any(dr < 0.2, axis=2)
el = el[el_keep]
```

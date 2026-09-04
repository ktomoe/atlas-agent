# ATLAS PHYSLITE Analysis Reference Index

## Purpose
This sub directory contains shared reference information for ATLAS PHYSLITE analyses.
Objects in PHYSLITE are already calibrated, so no CP tools are required.

## Provenance
Every branch name, type, `m_persKey` and "not readable by uproot" statement in
this directory was **checked against ATLAS Open Data (Run 2) files**, not taken
from documentation. Verified 2026-09-01 with uproot 5.7.6 and awkward 2.13.0.
It does not guarantee anything about the specific file at hand, nor is it intended to.

## Reference map
| Topic | Reference | Keywords |
|---|---|---|
| Reading PHYSLITE files (uproot) | [io.md](io.md) | Concatenate vs iterate; ElementLink split vs not-split|
| Event variables and examples | [events.md](events.md) | data or MC; GRL / lumiBlock|
| Electron variables and examples | [electrons.md](electrons.md) ||
| Photon variables and examples | [photons.md](photons.md) ||
| Muon variables and examples | [muons.md](muons.md) ||
| Tau variables and examples | [taus.md](taus.md) ||
| Jet variables and examples | [jets.md](jets.md) ||
| Track variables and examples | [tracks.md](tracks.md) | d0 significance; z0 sin(theta) |
| Vertex variables and examples | [vertices.md](vertices.md) | Hardest primary vertex |
| Trigger variables and examples | [triggers.md](triggers.md) | Trigger matching |
| Trigger advanced examples | [triggers-advanced.md](triggers-advanced.md) | Prescale; HLT decode|
| Cluster variables and examples | [clusters.md](clusters.md) ||
| Truth variables and examples | [truths.md](truths.md) ||
| Kinematic treatment | [kinematics.md](kinematics.md) | Invariant mass from four-vectors; overlap removal|
| Key hash values (m_persKey) | [keyhash.md](keyhash.md) |Resolve an unknown `m_persKey`|
| Metadata for Run2 Open data | [run2opendata.md](run2opendata.md) ||
| Vector optimization examples (awkward) | [vectorized-recipes.md](vectorized-recipes.md) | ElementLink gathering; option-type pitfalls; vectorized ΔR and overlap removal |

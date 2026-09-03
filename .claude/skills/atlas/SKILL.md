---
name: atlas
description: >-
  ATLAS PHYSLITE (DAOD_PHYSLITE) reference for uproot analysis: container and
  branch names (AnalysisMuons, AnalysisElectrons, InDetTrackParticles,
  GSFTrackParticles, PrimaryVertices, EventInfo, egammaClusters,
  AnalysisTrigMatch_HLT_*), with types, units and the selection recipes that
  are easy to get silently wrong. Use when reading PHYSLITE/xAOD ROOT files,
  resolving an AuxDyn branch name, selecting leptons by quality/ID/isolation,
  cutting on d0 significance or z0*sin(theta), following an ElementLink
  (m_persKey/m_persIndex) to tracks, clusters or truth particles, requiring a
  trigger by matching or prescale, applying a Good Run List, building an
  invariant mass or dR, vectorizing these with awkward, or looking up Run 2
  Open Data metadata (cross section, filter efficiency, k-factor, sum of
  weights, luminosity) for MC normalisation. Also ATLAS coordinates, helix
  parameters, particle masses. Not Athena/AnalysisBase; no jets, MET, taus,
  systematics.
---
# ATLAS Analysis Reference Index

## Purpose
This directory contains shared reference information for ATLAS analyses.
These references provide individual pieces of information and deliberately
omit end-to-end examples and scripts, and any prescribed order for reading them.
That is a design decision, not an omission. How these building blocks are
combined is specific to each analysis and must be defined by the analysis
that uses them.

## Rules
These hold for every analysis built from these references.

* **Selection values are the analysis's own.** Working points, thresholds and
  trigger choices marked *"not an official ATLAS working point"* show the
  mechanics of a cut, not a recommendation. Take the values from the analysis
  being processed.

* **Units are MeV and mm.** Every energy, momentum and mass branch is in MeV,
  every distance in mm. Convert to GeV only where a result is written out or
  plotted — never inside a selection, where the cut silently moves by 1000x.

* **Pseudo code is a specification, not an implementation.** The examples state
  *what* is required of one object. They are not the code to ship: translate
  them to the form the dataset size demands.

## Reference map
| Topic | Reference |
|---|---|
| Definition of coordinates | [references/coordinates.md](references/coordinates.md) |
| Physics parameters | [references/parameters.md](references/parameters.md) |
| Pseudo code rules | [references/pseudocode.md](references/pseudocode.md) |
| PHYSLITE analyses | [references/physlite/index.md](references/physlite/index.md) |

# Four-momentum and invariant mass
Containers store objects as (pt, eta, phi, m). Invariant masses are built by
summing **four-momenta** — never by combining masses or pT values directly.

## From (pt, η, φ, m) to (px, py, pz, E)
| Component | Expression |
|---|---|
| px | pt*cosφ |
| py | pt*sinφ |
| pz | pt*sinhη |
| E  | sqrt(px² + py² + pz² + m²) |

## Invariant mass of a set of objects
```python
E = px = py = pz = 0.0

for pt, eta, phi, mass in objects:
    E  += math.sqrt((pt*math.cosh(eta))**2 + mass**2)
    px += pt*math.cos(phi)
    py += pt*math.sin(phi)
    pz += pt*math.sinh(eta)

m2 = E**2 - (px**2 + py**2 + pz**2)
mass = math.sqrt(max(m2, 0.0))   # clamp: rounding can push m2 slightly below 0
```

# ∆R between two containers
```python
el_eta = event['AnalysisElectronsAuxDyn.eta'][ii]
el_phi = event['AnalysisElectronsAuxDyn.phi'][ii]
mu_eta = event['AnalysisMuonsAuxDyn.eta'][jj]
mu_phi = event['AnalysisMuonsAuxDyn.phi'][jj]

dphi = el_phi - mu_phi
dphi = (dphi + math.pi) % (2*math.pi) - math.pi   # See ../coordinates.md
deta = el_eta - mu_eta

dR = math.sqrt(deta**2 + dphi**2)
```
Within **one** container, skip the self pair (`ii == jj`), which has ∆R = 0.

## Overlap removal
**The code below is an example, not an official ATLAS working point**
Remove electrons, Keep muons with ∆R = 0.2.
```python
for ii in good_electrons:
    overlaps = False

    for jj in good_muons:
        dR = SEE_ABOVE
        if dR < 0.2:
            overlaps = True
            break

    if overlaps:
        continue # Remove ii electron
```

# Definition of ATLAS coordinates
Right-handed system with the origin at the interaction point: z along the beam line,
x towards the centre of the LHC ring, y upwards. The x-y plane is the transverse plane;
pT is the momentum measured in it, often in r-ϕ coordinates: ϕ is the azimuthal angle
from the x-axis around the beam line, r the distance from the beam line. θ is the polar
angle from the positive z-axis, and pseudorapidity η=−ln(tan(θ/2)).

## Side A and side C
**Side A is z > 0 (η > 0), side C is z < 0 (η < 0).** Region labels and branch names
carrying these letters mean exactly the sign of η — there is nothing to convert; the
geographical direction of side A only fixes the naming.

## ∆R and the ∆φ wrap
**Wrap ∆ϕ into [−π, π) before squaring it.** ϕ is stored in (−π, π], so a raw
subtraction lands in (−2π, 2π): a collinear pair with ϕ_a = 3.13 and ϕ_b = −3.13 gives
∆ϕ ≈ 6.26 instead of ≈ −0.02, and ∆R cuts, overlap removal and ∆R matching fail with
no error raised. A back-to-back pair is **not** a failure: wrapping only flips the sign
of ∆ϕ ≈ ±π, |∆ϕ| stays π.

```python
dphi = (phi_a - phi_b + math.pi) % (2*math.pi) - math.pi   # -> [-pi, pi)
dR = math.sqrt(deta**2 + dphi**2)
```

## Track helix parameters
Five parameters of a helix in a uniform magnetic field along z, **in the order they are
stored** in the track containers:

| Index | Parameter | Definition | Unit |
|---|---|---|---|
| 0 | d0 | Transverse impact parameter: transverse distance to the beamspot at the point of closest approach. | mm |
| 1 | z0 | Longitudinal impact parameter: z of the track at the point of closest approach, **measured from the reference point** (`vz` in physlite/tracks.md), not absolute z. | mm |
| 2 | ϕ | Azimuthal angle of the momentum, tanϕ ≡ py/px. | rad |
| 3 | θ | Polar angle of the momentum from the positive z-axis. | rad |
| 4 | q/p | Charge divided by the momentum magnitude. | 1/MeV |

cotθ ≡ pz/pT = 1/tanθ and 1/pT = |q/p|/sinθ appear in the literature; they are derived
quantities, **not** separately stored branches.

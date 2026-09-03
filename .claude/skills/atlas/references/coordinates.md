# Definition of ATLAS coordinates
A common **right-handed** coordinate system is used throughout the ATLAS detector. The interaction
point is defined as the origin of the coordinate system, and the beam line is defined as z-axis.
The positive x-axis points to the center of the LHC ring and the positive y-axis points upwards;
right-handedness then fixes the positive z-axis.
The x-y plane is referred to as the transverse plane to the beam axis. Particle momentum measured
in the transverse plane is referred to as the transverse momentum, pT. The transverse plane is often
described in terms of r-ϕ coordinates. The azimuthal angle ϕ is measured from the x-axis around the
beam line. The radial dimension, r, measures the distance from the beam line. The polar angle θ is
defined as the angle from the positive z-axis, then pseudorapidity η is defined as η=−ln(tan(θ/2)).

## Side A and side C
The two halves of the detector are named after the sign of z: **side A is z > 0, side C is z < 0**,
so η > 0 is side A and η < 0 is side C. Detector-region labels and some branch names carry these
letters, and they are the same statement as the sign of η — there is nothing to convert. What an
analysis needs is this sign convention; the geographical direction of side A only fixes the naming,
and is stated differently in different references.

## ∆R and the ∆φ wrap
**∆ϕ must be wrapped into an interval of length 2π around 0 before it is
squared.** ϕ is stored in (−π, π], so a raw subtraction lands anywhere in
(−2π, 2π). The failure is at the ±π boundary: a collinear pair with ϕ_a = 3.13
and ϕ_b = −3.13 gives ∆ϕ ≈ 6.26 instead of ≈ −0.02, so two objects that overlap
are reported at ∆R ≈ 6.3. ∆R cuts, overlap removal and ∆R matching then fail
with no error raised. A back-to-back pair is **not** a failure: ∆ϕ ≈ ±π is the
correct answer there, and wrapping only flips its sign — |∆ϕ| stays π.

```python
dphi = phi_a - phi_b
dphi = (dphi + math.pi) % (2*math.pi) - math.pi   # -> [-pi, pi)
dR = math.sqrt(deta**2 + dphi**2)
```

## Track helix parameters
Trajectories of charged particles can be described by five helix parameters in an ideal uniform
magnetic field parallel to the z-axis.

They are listed below **in the order they are stored** in the track containers.

| Index | Parameter | Definition | Unit |
|---|---|---|---|
| 0 | d0 | Transverse impact parameter, defined as the transverse distance to the beamspot at the point of closest approach. | mm |
| 1 | z0 | Longitudinal impact parameter, defined as the z position of the track at the point of closest approach, **measured from the reference point** (see `vz` in physlite/tracks.md), not in absolute z. | mm |
| 2 | ϕ | Azimuthal angle of momentum direction in the transverse plane, where tanϕ ≡ py/px. | rad |
| 3 | θ | Polar angle of momentum direction, measured from the positive z-axis. | rad |
| 4 | q/p | Charge divided by the magnitude of the momentum. | 1/MeV |

Two equivalent forms of the last two parameters appear in the literature; they are
derived quantities, **not** separately stored branches:

* cotθ ≡ pz/pT = 1/tanθ, the cotangent of the polar angle.
* 1/pT, the reciprocal of the transverse momentum, = |q/p| / sinθ.

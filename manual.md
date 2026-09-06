# GeoEQ Cap technical manual

## Verification benchmark: shallow strip-footing bearing resistance

**Document status:** reproducible numerical benchmark  
**Software:** GeoEQ Cap finite-element limit analysis (FELA)  
**Benchmark date:** 8 August 2026  
**Reproduction script:** `scratch/verify_design_frameworks.py`

> This benchmark verifies the FELA collapse-resistance interval and the
> arithmetic used by GeoEQ Cap's design frameworks. It does not establish
> complete Eurocode, AASHTO or IBC compliance, and it does not verify
> settlement or any other serviceability limit state.

### 1. Purpose

The benchmark checks three independent parts of the calculation:

1. the lower and upper FELA solutions must contain a known analytical collapse
   load;
2. the independently bundled method-of-characteristics reference must
   reproduce the same analytical result; and
3. every design framework must transform the nominal resistance without
   changing or mislabelling the underlying numerical result.

The selected problem is Prandtl's surface strip footing on weightless,
homogeneous, purely cohesive soil. It is especially useful because its exact
plasticity solution is known. Hjiaj et al. identify Prandtl's `Nc` and
Reissner's `Nq` as exact for a strip footing on weightless soil, and published
FELA studies routinely use the same problem as a verification benchmark.

### 2. Analytical ground truth

For a surface strip footing in undrained soil with `phi = 0`, zero surcharge
and negligible unit weight,

```text
Nc    = 2 + pi = 5.141592654
q_ult = Nc cu
```

Using `cu = 50 kPa`,

```text
q_ult,exact = (2 + pi)(50) = 257.079633 kPa
```

This value is external to GeoEQ Cap: it is calculated directly from the
closed-form Prandtl solution, not from either FELA solver.

### 3. Numerical model

| Item | Benchmark value |
|---|---:|
| Formulation | Plane strain |
| Foundation | Rough rigid strip footing |
| Footing width `B` | 2.0 m |
| Embedment `Df` | 0.0 m |
| Soil profile | Homogeneous undrained clay |
| Undrained strength `cu` | 50.0 kPa |
| Friction angle `phi` | 0 degrees |
| Unit weight | `1e-6 kN/m3` (numerically weightless) |
| Surcharge | 0 kPa |
| Groundwater | None |
| Analysis domain | 40 m wide x 20 m deep |
| Symmetry | Half model |
| Mesh | 1,102 triangles |
| Footing-edge refinement | 4.0x |
| Solution | Lower and upper bounds; best valid upper element |

The tiny positive unit weight avoids a nonphysical zero material property
while changing the analytical pressure by a negligible amount.

### 4. Acceptance criteria

For each framework, the independently transformed analytical value must obey

```text
q_design,lower <= q_design,exact <= q_design,upper
```

Additional checks require that:

- the lower and upper results retain their correct ordering;
- AASHTO and US-building wrappers leave the nominal FELA interval unchanged;
- EC7 DA1 evaluates both combinations and identifies the governing one;
- the custom material factor reaches the solver before the output resistance
  factor is applied; and
- the method-of-characteristics result agrees with the closed-form value to
  within `0.05 kPa`.

The interval midpoint is reported only as an accuracy observation. It is not
used as the acceptance criterion and is not a certified design value.

### 5. Results

| Design framework | Independent analytical value (kPa) | GeoEQ lower (kPa) | GeoEQ upper (kPa) | Midpoint error |
|---|---:|---:|---:|---:|
| Analysis only — nominal ultimate resistance | 257.080 | 242.198 | 273.285 | +0.26% |
| Eurocode 7 — DA1 governing | 183.628 | 172.998 | 195.203 | +0.26% |
| AASHTO LRFD format — `phi = 0.45` | 115.686 | 108.989 | 122.978 | +0.26% |
| US calculated allowable bearing — `FS = 3.0` | 85.693 | 80.733 | 91.095 | +0.26% |
| User-defined — `gamma_cu = 1.25`, `gamma_R = 1.40` | 146.903 | 138.399 | 156.163 | +0.26% |

Every independently calculated value lies inside the corresponding GeoEQ Cap
interval.

The method-of-characteristics engine returned:

```text
q_ult,MOC = 257.080 kPa
q_ult,exact = 257.080 kPa
```

The agreement is within the displayed precision and provides a second check
independent of the lower- and upper-bound finite-element formulations.

### 6. Framework calculations

#### 6.1 Analysis only

No code factors are applied:

```text
q_exact = 257.079633 kPa
```

The result is an ultimate collapse resistance, not an allowable pressure.

#### 6.2 Eurocode 7 DA1

For this purely cohesive benchmark, DA1-2 applies the recommended M2 divisor
`gamma_cu = 1.4`. DA1-1 retains the characteristic strength, so DA1-2 governs:

```text
q_design,exact = 257.079633 / 1.4
               = 183.628309 kPa
```

The application runs both DA1 combinations rather than assuming which one
governs. Nationally Determined Parameters must still be checked against the
project National Annex.

#### 6.3 AASHTO LRFD format

An illustrative, explicitly confirmed project factor `phi = 0.45` is used:

```text
phi Rn = 0.45(257.079633)
       = 115.685835 kPa
```

The nominal FELA interval is unchanged before multiplication. The value `0.45`
is used because it also appears in FHWA's published spread-footing example;
it is not a universal AASHTO factor and must not be silently transferred to a
different method, limit state, agency or project.

#### 6.4 US building calculated allowable bearing

An illustrative project-confirmed global factor `FS = 3.0` is used:

```text
q_allowable,exact = 257.079633 / 3.0
                  = 85.693211 kPa
```

This is a calculated allowable pressure. It is separate from the presumptive
vertical foundation pressures in IBC Table 1806.2 and does not replace the
geotechnical investigation or settlement assessment required for the project.

#### 6.5 User-defined factors

The benchmark first divides undrained strength by `gamma_cu = 1.25` inside the
FELA model, then divides the verified resistance by `gamma_R = 1.40`:

```text
q_design,exact = 257.079633 / (1.25 x 1.40)
               = 146.902647 kPa
```

This proves that material and resistance factors are applied at different,
auditable stages. User-defined factors are not a published standard.

### 7. Interpretation

The common relative interval and midpoint error after every conversion show
that the design wrappers do not distort the numerical solution. EC7 and the
custom partial-factor case correctly modify the model supplied to FELA;
AASHTO and US-building formats correctly operate on the unchanged nominal
resistance.

This benchmark supports the following limited claim:

> For the stated Prandtl idealisation, GeoEQ Cap brackets the exact collapse
> resistance and correctly applies the declared design-framework factors.

It does **not** support these broader claims:

- every soil profile or footing geometry is thereby validated;
- an illustrative `phi` or `FS` is universally applicable;
- the result is a complete foundation design;
- settlement, sliding, overturning, overall stability, scour, seismic
  performance or structural footing resistance has been verified; or
- a particular authority must accept FELA in place of its prescribed nominal
  resistance method.

### 8. Reproduction

From the repository root in PowerShell:

```powershell
$env:QT_QPA_PLATFORM = "offscreen"
& .\.venv\Scripts\python.exe .\scratch\verify_design_frameworks.py
```

The test exits with a nonzero status if an analytical value falls outside its
reported interval, a wrapper changes the nominal solution unexpectedly, EC7
selects the wrong governing combination, or the method-of-characteristics
reference departs from the exact value.

For a published result, record the Git commit, dependency versions, operating
system and complete console output with the report. The benchmark values in
this chapter were generated from the working tree based on commit
`08fd0e10f042`; publication should reference the later commit containing this
manual and test.

### 9. References

1. Prandtl, L. (1920/1921). Classical analytical solution for the plastic
   bearing capacity of a strip footing on a weightless half-space.
2. Hjiaj, M., Lyamin, A.V. and Sloan, S.W. (2005). “Numerical limit analysis
   solutions for the bearing capacity factor N-gamma.” *International Journal
   of Solids and Structures*, 42, 1681–1704.
   <https://www.newcastle.edu.au/__data/assets/pdf_file/0006/22596/72_Numerical-limit-analysis-solutions-for-the-bearing-capacity-factor-N-gamma.pdf>
3. ISSMGE (2013). Published FELA strip-footing benchmark reporting the Prandtl
   value and numerical lower/upper bounds.
   <https://www.issmge.org/uploads/publications/1/2/731-734.pdf>
4. European Commission Joint Research Centre. “Shallow foundations: design of
   spread foundations,” EN 1997 training material.
   <https://eurocodes.jrc.ec.europa.eu/publications/shallow-foundations-design-spread-foundations>
5. Federal Highway Administration. *Selection of Spread Footings on Soils to
   Support Highway Bridge Structures*, FHWA-RCTD-10-001, Appendix E.
   <https://highways.fhwa.dot.gov/sites/fhwa.dot.gov/files/FHWA-RCTD-10-001.pdf>
6. International Code Council. *2024 International Building Code*, Chapter 18,
   Soils and Foundations.
   <https://codes.iccsafe.org/content/IBC2024V2.0/chapter-18-soils-and-foundations>

## Automatic numerical sensitivity and convergence study

Run the baseline model first. In **Calculate** or **Output**, choose **Run
verification study**. The study works on serialized copies; it never resizes
the open project, replaces its mesh, or moves its layer interfaces.

The presets control safety limits, not a fixed case list:

- **Quick**: automatic screening with smaller trial and element limits.
- **Engineering convergence**: the recommended design-model review.
- **Research**: more domain trials, adaptive passes, and a larger element budget.

The sequence is automatic. It increases width until side containment and
successive-bound stability pass, then increases depth until bottom containment
and stability pass. Mesh levels are refined on that domain. If needed,
failure-zone adaptive passes continue until acceptance or a declared
trial/pass/element limit. Every completed trial is retained.

The verdict requires all of the following:

1. the change of both certified bounds from the preceding trial is within the
   selected stability tolerance;
2. separate side and bottom mechanism responses are below the selected limit;
3. the accepted mesh meets the maximum lower–upper bound-gap limit; and
4. no required solver, trial-count, pass, or element limit was reached.

When these pass, GeoEQ reports the minimum accepted width/depth, mesh class and
element count, and certified bearing-capacity interval. Otherwise it states
that convergence was not demonstrated and gives the next enlargement or
refinement action. An unaccepted model is never silently presented as safe.

`H/B` cases are deliberately excluded from that verdict. Moving a soil-layer
interface changes the physical idealisation; it is a parameter-sensitivity
study, not evidence of numerical convergence.

The results window leads with the accepted final-analysis model or a clearly
unaccepted last-tested model and specific next action. It includes the complete
trial record and publication plots against `W/B`, `D/B`, element count, and
boundary response. CSV/JSON retain the domain, mesh, both bounds, bound gap,
successive change, boundary response, criterion, runtime, and status. The active
plot exports to PNG/SVG. A completed study is frozen onto the
baseline calculation and included automatically in HTML/PDF and plain-text
reports. Editing or recalculating the model invalidates it. Cancellation is
safe between cases; the current native solver case is allowed to finish.

When **Run another study** keeps the same model and options, completed cases
are reused. This resumes a cancelled partial study without repeating its
finished solves. Changing the preset or acceptance settings starts a fresh
audit record.

Reproduce the focused workflow checks with:

```powershell
$env:QT_QPA_PLATFORM = "offscreen"
& .\.venv\Scripts\python.exe .\scratch\verify_verification_study.py
```

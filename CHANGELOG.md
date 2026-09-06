# Changelog

All notable changes to GeoEQ Cap are documented here.
Dates use YYYY-MM-DD.

## 1.0.0 — 2026-09-05

First public release. Version 1.x is free.

### What it does

- **Bearing capacity by finite element limit analysis (FELA).** Every
  analysis returns a *lower* and an *upper* bound on the collapse load.
  Each bound is checked independently after the solve — equilibrium and
  yield for the lower bound, the flow rule and dissipation for the upper
  bound — so the reported interval is one the software has verified, not
  one it has assumed.
- **Strip and circular footings.** Plane-strain (strip) results are
  rigorous bounds. Circular footings are solved as an axisymmetric
  problem and reported as estimated (near) bounds, validated against the
  exact circular solutions in the literature.
- **Layered soil, groundwater and surcharge.** Any number of Mohr–Coulomb
  layers, drained or undrained, with a water table and a surface
  surcharge. Slip is allowed along layer interfaces at the correct
  strength of each side.
- **Inclined and eccentric loads; V–H and V–M envelopes.** Sweep the load
  direction to trace the combinations of vertical load, horizontal load
  and moment the ground can carry.
- **Adaptive mesh refinement** narrows the gap between the two bounds
  where the failure mechanism forms.
- **Results you can look at.** Collapse mechanism, failure band, stresses,
  strength mobilisation and velocities as filled contours, with the same
  presentation choices reviewers expect: banded or smooth, full or robust
  colour range, half or whole model.
- **Comparison with conventional methods.** The FELA interval alongside
  the classical bearing-capacity formulas and a method-of-characteristics
  solution, in one chart.
- **A calculation report ready for review**, as PDF: cover with document
  identity and a verification seal, numbered sections, assumptions,
  measured-versus-limit QA checklist, figures, and a verbatim record of
  every solver note in an appendix.
- **Design formats.** The characteristic resistance can be presented
  in an Eurocode 7 partial-factor format or converted to LRFD / ASD
  values, always stated explicitly on the report.
- **Automatic verification studies.** Domain-size, mesh and layer
  studies run on copies of the project and report whether the answer is
  converged.
- **Boundary conditions drawn on the model**, as in the FE packages you
  already know.
- **Offline-first.** Projects are plain `.gec` files on your disk. No
  account. Automatic update notification when online.

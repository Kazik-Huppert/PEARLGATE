# PEARLGATE

**Beam-target propulsion for interstellar flight — physics model, design-space explorer, and ship design tools.**

PEARLGATE is a research programme into beam-target propulsion (BTP): a high-energy particle beam is
generated and fired *onboard* the spacecraft into a high-Z condensed-phase target, the resulting
hadronic cascade (spallation, fission-fusion, pion/kaon showers) is directed out the back, and the
reaction products are used **directly as thrust**. No prior propulsion system does all three of those
things at once — ADS designs (Rubbia, MYRRHA) use the beam and the target but treat the products as
shielding and waste; ICF pulse ships such as *Daedalus* use onboard beams but fire at low-Z targets,
so there is no hadronic cascade.

Everything here is a single-file HTML tool. No build step, no dependencies to install, no server
required — open a file in a browser.

---

## Contents

| File | What it is |
|---|---|
| [`index.html`](index.html) | **The 0-D design-space explorer.** Live parameter sliders driving the full propulsion cascade: mass partition, relativistic fragment physics, ablator jet, thermal management, power budget, antimatter thresholds, ISM harvesting, and an animated mission profile. |
| [`0D.html`](0D.html) | **SDS-FAR Mission Physics Companion v2.** The written derivation behind the simulator — governing equations section by section, parameter rationale, what changed in v2, the PCT materials reference, the BTP category definition, and an explicit statement of what is *not* modelled. Static document, no script. |
| [`LIC 2.html`](LIC%202.html) | **Local Interstellar Cloud.** A 3D map of 191 nearby star systems (39 flagged habitable). Click a star to launch a ship; journeys accumulate in a log. |
| [`pearlgate.html`](pearlgate.html) | **PEARL-GATE Freighter.** Ship and habitat geometry builder — compressed-candle engine, spine, habitat, cargo tiers, radiators — with a library of pinned mission presets for comparison against other designs. |

> Filename note: `LIC 2.html` contains a space, so raw links to it need `%20` escaping. GitHub's file
> browser handles it; `git` and shell scripts need quoting.

---

## Quick start

Open `index.html` in any modern browser. That's the main tool.

```
git clone https://github.com/Kazik-Huppert/PEARLGATE.git
cd PEARLGATE
start index.html          # Windows
xdg-open index.html       # Linux
open index.html           # macOS
```

Or just browse to the raw file on GitHub and use your browser's view-source. Every tool is
self-contained.

### Network requirements

None of these run fully offline out of the box — each pulls a CDN dependency on first load:

| File | Dependency |
|---|---|
| `index.html` | three.js r128 (cdnjs) |
| `pearlgate.html` | three.js r128 (cdnjs) |
| `LIC 2.html` | three.js 0.128.0 + `OrbitControls` + `CSS2DRenderer` (unpkg, ES modules), plus an ESO Milky Way texture and a background image from an image host |
| `0D.html` | Google Fonts (IBM Plex) |

`LIC 2.html` is the most network-dependent: it uses an ES module import map, so it also needs to be
served over `http://` rather than opened as a bare `file://` path in some browsers. If the module
import is refused, run a one-line static server from the repo root:

```
python -m http.server 8000     # then visit http://localhost:8000/LIC%202.html
```

---

## The 0-D explorer (`index.html`)

### The idea

Six linked sections, recomputed on every slider move:

1. **Mission trajectory** — destination distance, arrival speed, target transit time
2. **Mass** — dry mass, mass ratio, wet mass, and the split between nuclear fuel, graphene ablator, and trapped fragments
3. **Propulsion physics** — fragment kinetic energy, ablator jet, combined exhaust velocity, Δv, Isp
4. **Power & thermal management** — waste heat, thermoelectric recovery, radiator sizing, the beam power budget, antimatter thresholds
5. **ISM harvesting** — interstellar-medium feedstock collection during coast
6. **Animation** — time warp and a WebGL view of the engine burning

Every input is a slider bounded by a number box you can click to retype, so you can push a
parameter past the default range rather than being boxed in.

### The physics, condensed

Trapped fragments `(1−ε)·ṁ_nuc` are expelled by the ablator jet:

```
jet mass flow  = (K + 1 − ε)·ṁ_nuc
energy per kg  = η·Q_ff / (K + 1 − ε)          Q_ff already contains (1 − ε)
v_p            = √(2·(E_ab_mix − H_sub))
v_eff          = (ε·[γ]·v_ff + (K + 1 − ε)·v_p) / (1 + K)
Q_ff           = (1 − ε)·(γ − 1)·c²            (relativistic)
Δv             = v_eff·ln(R)   or   c·tanh(v_eff/c · ln R)
```

`K` is the graphene-ablator to nuclear-fuel mass ratio. `K = 0` gives a pure FFRE (fission fragment
rocket exhaust); `K > 0` engages the ablating drive. `η` is *derived*, not typed in:

```
Π₆ = K / (1 − ε)        η = η_geo · min(1, Π₆ / Π₆_s)
```

More ablator relative to trapped fragments means better coupling, less waste heat, and a lighter
radiator. `Π₆_s` is the coupling saturation point — the value of `Π₆` at which the ablator fully soaks
the trapped heat.

The recommended trajectory is **Sprint → Coast → Break**, with the cruise speed at the flip set to
`(Δv + v_arr)/2`. The solver resolves three cases automatically: sprint-only (destination reached
mid-burn, reached by binary search on cruise speed), full-burn coast, and a normal
accelerate/coast/decelerate profile.

### Defensibility colouring

This is the part that makes the tool worth using. Every input is colour-coded by how close it sits
to something real:

- **white** — demonstrated or comfortably within range
- **amber** — research target: physically motivated, and a development path exists
- **orange** — fantasy: no known near-term path to this value
- **red** — the physics breaks (e.g. `E_ab < H_sub`, so the jet quenches)

Hover any coloured value for the specific reason. The colour scale is applied per-parameter with
explicit thresholds — `k_eff` above 0.98 is amber, above 0.995 is orange; `ZT` above 2.5 amber, above
3.0 orange; and so on.

### Mission presets

The `⇄ Proxima` button cycles three fully-specified Proxima Centauri rendezvous configurations:

| Preset | Transit | Character |
|---|---|---|
| **Buildable Now** | 284 yr | Every input white. TEG-only, isotropic ε = 0.50, no collimation, nothing unbuilt. |
| **Research Target** | 129 yr | Zero orange inputs — every parameter amber or white. The genuine all-amber floor. |
| **Aspirational** | 55 yr | Only three orange inputs: an rSM 150 kg/MW radiator, ε = 0.97 magnetic collimation, and a 500 MW beam. |

Each carries a note on what the unlock was and what it cost. The point of the middle preset is the
finding: **sub-100-year Proxima rendezvous is not reachable without at least one orange input.**

### Animation modes

- **CLOCK** — mission time advances by real elapsed time × warp; frame-rate dependent
- **PHYSICS** — advances by a fixed `1/60 s` step × warp; repeatable
- **γ RELATIVISTIC / ∿ CLASSICAL** — toggles `Q_ff = (γ−1)c²`, `F = ½ṁγv`, `Δv = c·tanh(v_eff/c·ln R)`
  against the classical `v²/4`, `½ṁv`, `v_eff·ln R`

The 3D view is WebGL: the candle burns down as propellant depletes, beam particles and ISM particles
stream through, and the thrust sphere and locus flash during the burn phases.

---

## The physics companion (`0D.html`)

The derivation behind the simulator, dated May 2026. Ten sections, written to be checked:

- **§1–2** — what SDS-FAR models, then the governing equations: mass partition, fragment physics,
  ablator jet, combined exhaust velocity, mission Δv, thrust, the subcritical power budget, PCT
  thermoelectrics, and the self-sustaining ratio
- **§3** — what changed in v2, and why
- **§4** — default parameters with physical rationale and permitted ranges
- **§5** — performance at defaults, for cross-checking against the live simulator
- **§6** — why the rapidity correction matters. At 4%c it adds ~0.07 km/s to `v_eff`, which
  propagates through the rocket equation into a **one-day** difference in transit time to 550 AU.
- **§7** — PCT material reference table (Bi₂Te₃, clathrates, half-Heuslers, topological insulators)
- **§8** — the BTP category definition and prior-art analysis
- **§9** — six features of the full architecture still to be modelled, each with its physics,
  simulator impact, and complexity rating: Double Punch wakefield boost, ISM funnel collection,
  stage-dependent exhaust velocity, ECM threshold cascade channels, UV seeding of the ablator
  plasma, and liquid droplet radiator overflow
- **§10** — limitations, stated plainly

---

## Local Interstellar Cloud (`LIC 2.html`)

191 nearby systems, 39 marked habitable, coloured by visual magnitude. Drag to orbit, scroll to zoom,
click a star to open its panel and launch. `SHIP_SPEED` and `TIME_PER_FRAME` are set at the top of the
script if you want to change how fast the colony ship crosses the map.

---

## Freighter builder (`pearlgate.html`)

Build a ship and pin the numbers that matter, then let the tool derive the rest. Two mission
profiles — **burn-coast** and **rendezvous** — with ten pinnable constraint variables (distance,
transit time, average speed, burn fraction, Δv, peak speed, exhaust velocity, dry mass, wet mass,
propellant mass). Pin what you know, and the remaining figures solve.

Geometry is parametric: habitat radius, arm pin radius, arm slenderness, burn cycle, shot cadence,
mass per shot, flash intensity, berths, nausea ceiling, module height, container type, beam tube
radius, cargo tiers and bays (heavy / bulk / precision), module density, and a fusion secondary
characterised by compression ratio `C` and `φ_in`.

Toggles: habitat spin, thin-film radiators, labels, candle burn, capture beam, freeze burn, and
light/heat colouring.

**Pinned mission presets:**

*Interstellar:* NSWR, NSW Comet, Interstellar Orion, **PEARL-GATE 42 t**, **PEARL-GATE 980 t**
*For comparison:* *Daedalus* (BIS 1973–78), *Icarus Firefly* (BIS 2009–23), *Longshot* (US Naval
Academy / NASA 1987–88)
*Round trip:* Moon, Venus, Mars, Mercury, Asteroid, Jupiter

Each preset loads its profile and pinned values and carries a note explaining the source figures and
what was derived versus what was given. The *Daedalus* preset is a good self-check: it pins a
published wet mass and derived dry mass, and returns 980 t.

---

## Honest scope

This is a **feasibility exploration framework**, not an engineering design tool. The model conserves
mass, energy, and momentum within its 0-D framework, and has no free parameters beyond the sliders you
set. What it cannot do, in the author's own list:

- **Spatial** — no temperature gradient inside the ablator, no beam profile, no plasma density distribution
- **Stability** — no plasma instability modelling, no neutron transport, no criticality transients
- **Structures** — no transient thermal-structural coupling, no ablator erosion geometry
- **Radiation** — no full relativistic particle cascade simulation, no radiation damage accumulation
- **Fluids** — no CFD, no magnetohydrodynamics, no plasma sheath modelling

Two further caveats stated in the model header: the wakefield boost `η_wake` is an **external
velocity multiplier** applied to both exhaust streams, representing speculative energy injection, and
the MHD direct-conversion tap is treated as drawing from directed charged exhaust. Both are flagged
as such in the interface.

Wakefield, ISM, and antimatter contributions are all speculative and are marked as such. A 0-D model
is the right foundation for exposing the amplification chain and mapping parameter space; it is not a
substitute for 1-D/2-D MHD, PIC, or neutronics codes.

---

## Patent

**GB2522564.0A** — inventor Kazik Huppert, International Space Engineering Academy
(`DrKH@ispacee.org`). The prior-art analysis and BTP category definition are the intellectual
property of the inventor.

---

## Repository notes

- No licence file is present. All rights reserved by default — add one before redistributing.
- No build tooling, package manager, tests, or CI. The JS in each file is self-contained.
- 145 commits, `main` only, no releases or tags.
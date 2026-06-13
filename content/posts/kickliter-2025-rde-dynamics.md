---
title: "Paper Review: Asymptotic and Transient Dynamics of Rotating Detonation Engines"
date: 2026-06-13
draft: false
math: true
tags: ["RDE", "paper-review", "bifurcation-analysis", "CFD"]
summary: "Kickliter et al. use transient CFD and numerical continuation to map out the operating regimes of a 2D RDE model, and find that the primary instability is not what earlier reduced-order work suggested."
---

## Why this paper matters

Rotating detonation engines work by sending detonation waves spinning
azimuthally around an annular combustor instead of burning at roughly
constant pressure like a normal rocket engine. The reason people care is
pressure gain — RDEs theoretically extract more useful work from the same
propellants. The problem this paper attacks is that experiments and
simulations keep showing different wave behaviors at the same operating
conditions. One run gives you a single wave, another gives you
counter-rotating pairs, and computational studies have even shown up to 24
waves. It is not clear if this is from real configuration differences
between facilities or if multiple solutions actually exist at the same
operating point and different runs just land on different ones.

What makes this paper interesting to me is the approach. Most RDE
simulations take weeks to run on supercomputers, so nobody has been able
to systematically map out the solution space. The authors get around this
by combining two tools: transient CFD to find the operating regimes, and
numerical continuation (through their own tool LOCAFOAM) to precisely
locate the bifurcations between them. The continuation part is what I had
not seen applied to RDEs before, and it is what lets them trace stability
boundaries instead of just sampling them.

## The setup

One of the first things I noticed about how the authors set up this model
is that they did not try to simulate a full 3D engine. Instead, they used
a 2D "unwrapped" domain that is 22.3 cm by 7.62 cm to keep the computation
cheap enough to do a full parameter sweep.

They also made a lot of simplifying assumptions about the physics. The
reactants are assumed to be perfectly premixed, the combustion uses
single-step Arrhenius kinetics instead of detailed chemistry, and they
ignore viscous and thermal diffusion. One of the bigger simplifications is
that they decoupled the injection system from the chamber dynamics, which
real engines do not do. They did this by prescribing a constant inlet mass
flow rate and temperature at the bottom boundary.

For the other boundaries, the left and right sides are periodic to mimic
the annular loop of a real RDE, and the top is a partially reflecting
pressure outlet set to atmospheric conditions ($p_\infty = 0.1$). The two
main parameters they sweep are the dimensionless inlet mass flow rate
($\dot{m}$) and the Damköhler number ($Da$), which is the ratio of the
flow time-scale to the chemical time-scale.

## Key results

### The four operating regimes

The first big result is that when the authors swept across $Da$ and
$\dot{m}$ values, the system shows four distinct asymptotic modes: no
ignition, periodic explosions, intermittent waves, and steady traveling
waves. The steady traveling wave is the mode that RDEs are designed to
operate in, while the periodic explosion mode behaves almost identically
to the longitudinal pulsed detonation (LPD) instability that has been
seen in real engines.

### How Da and ṁ act the same way

The second result was that the system behavior is governed by the balance
between the flow time-scale and the chemical time-scale. Increasing
$\dot{m}$ shortens the time the flow spends in the chamber, which has a
nearly identical physical effect as decreasing $Da$, which lengthens the
chemical time-scale.

$$Da \propto \frac{\tau_{\text{flow}}}{\tau_{\text{chem}}}$$

Because $Da$ is this ratio, dialing $\dot{m}$ up or dialing $Da$ down both
lower the same effective ratio. This is why pushing either parameter to
its extreme causes lower frequency explosions or full blowoff.

### The Hopf bifurcation result

The last result was the numerical continuation finding. Using LOCAFOAM,
the authors solved the steady versions of the governing equations to map
the engine's stability without needing to time-step the system. They
found that the steady state branch loses stability to a supercritical
Hopf bifurcation, and this bifurcation corresponds directly to the
periodic explosion instability seen in the transient calculations. They
also showed that the critical $Da$ at which this happens is practically
insensitive to grid resolution, which is what makes the result physical
rather than numerical.


## My take


Overall, one of the things I took away from this paper was the assumptions
the experimentors made for this research and simulation regarding RDE
For example, they removed the chamber-injector coupling, used single-step
Arrhenius kinetics instead of detailed chemistry, and assume fully
premixed reactants. They wrote in the paper that the model is meant to generate
testable hypotheses rather than capture RDE behavior.

The second thing was that they found that the
primary instability of this system is the longitudinal pulsed detonation
mode, not the rotating wave. This goes directly against earlier 1-D
reduced-order work by Koch and Kutz, which had rotating detonations as
the primary instability. The authors talk about this and say generality is
future work. It is also talked about that the outlet boundary condition is doing a
lot of work in their result. Their partially reflecting pressure outlet
is what keeps the high-temperature pockets that trigger the periodic
explosions as shown in Fig 3. If that is right, then the primary-instability conclusion might affect the outlet conditions that look like an open downstream end rather than being a general RDE result.

One last thing that I noticed was the bistability result in Fig. 10.
At $(Da = 40, \dot{m} = 2.1)$, they show that a blast-kernel
initialization gives one wave, but a controlled two-wave initialization
with the reaction rate temporarily clamped gives two co-rotating waves.
This is direct numerical evidence that wave number is not set by
operating conditions alone, which is a pretty important finding. But it
sits in a single paragraph at the end of section IV.A. I think this
result deserved its own deeper treatment.

## What I would want to see next

One thing I would want to see in future work is what happens when the
single-step Arrhenius kinetics is replaced with multistep chemistry. The
current model cannot really distinguish between fuels that behave very
differently in real RDEs, like hydrogen versus methane.
Connecting the bifurcation analysis to actual propellant chemistry would
make these results a lot more useful for engine designers.

## References

1. Kickliter, T. M., Young, E., Acharya, V., and Lieuwen, T. C.,
   "Asymptotic and Transient Dynamics of Rotating Detonation Engines,"
   *AIAA SciTech 2025 Forum*, Orlando, FL, 6–10 January 2025, AIAA Paper
   2025-0399. [DOI](https://doi.org/10.2514/6.2025-0399)
2. Koch, J., and Kutz, J. N., "Modeling thermodynamic trends of rotating
   detonation engines," *Physics of Fluids*, Vol. 32, No. 126102, 2020.
3. Anand, V., St. George, A., and Gutmark, E., "Longitudinal pulsed
   detonation instability in a rotating detonation combustor,"
   *Experimental Thermal and Fluid Science*, Vol. 77, 2016, pp. 212–225.
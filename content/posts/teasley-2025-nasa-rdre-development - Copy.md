---
title: "Paper Review: NASA's Rotating Detonation Rocket Engine Development"
date: 2026-06-14
draft: false
math: true
tags: ["RDE", "paper-review", "NASA", "rocket-propulsion", "experimental"]
summary: "Teasley overviews NASA Marshall's RDRE test program, including the major finding that single-wave operation destroys hardware and the demonstration of detonations at 1250 psia."
---

## Why this paper matters

This paper is a program-level overview of where NASA Marshall actually is
on rotating detonation rocket engine development. Most of the RDE papers
I have read so far focus on simulation or small lab tests, so it was
useful to read something that lays out what is happening at the
engineering and integration level. The author, Thomas Teasley, leads the
RDRE development work at Marshall, so the paper covers the full picture
including hot-fire test results, hardware design, manufacturing
constraints, and the technology maturation plan going forward.

What stood out to me is that the paper is honest about which findings
were surprising and which design assumptions had to change. There are
also several results that directly contradict things I had read in
academic RDE papers, which I think is the most useful part of a paper
like this.

## The setup

Instead of one model or one test, the paper covers a whole program. NASA
Marshall has been running an Early Career Initiative project on RDREs
and has built two parametric test platforms. The first is MARLEN, a
subscale hardware platform designed for rapid component swaps to test
heat flux, Isp, and injector behavior. The second is SWORDFISH, a full
scale 10,000 lbf thrust chamber that lets them compare scaling between
the subscale and full scale results.

The paper also lays out what NASA considers the four critical technology
elements (CTEs) for an RDRE: the thrust chamber and cooling subsystem,
the injector, the nozzle extension, and the turbomachinery. Each one
has its own manufacturing process, alloy requirements, and test
objectives. Most of the chamber and injector hardware is built using
laser powder bed fusion (L-PBF) of GRCop-42 because no other alloy or
process has been shown to survive the heat fluxes at the operating
conditions they want.

The propellants tested so far include methane/oxygen, kerosene/oxygen,
and hydrogen/oxygen, with future plans to test liquid hydrocarbons with
air and hydrogen peroxide.

## Key results

### Single-wave operation destroys hardware

This was the result that surprised me the most. At both subscale and
full scale, they found that single-wave operation imparts a large
unbalanced vibratory load on the hardware. The thrust traces in Fig. 7
show this clearly — the single-wave case has a thrust signal that
oscillates much more violently than the 2-wave case for the same
average thrust. The damage from this includes inner body burn-through,
fused components around bolt holes, shattered metal seals, instrument
damage, and cracking of the exit duct.

This finding directly conflicts with how a lot of academic RDE work
treats wave number. In computational papers I have read, single-wave
operation is often presented as the cleanest or most desirable mode
because it is the simplest to model and analyze. Teasley's data shows
that single-wave operation is actually the mode you want to avoid in
flight hardware. Future RDRE designs will need to be intentionally
biased toward multi-wave operation, even when throttled down.

### Detonation at 1250 psia is possible

Before this test, the expectation was that high chamber pressures would
cause deflagration to dominate and erase the pressure-gain benefit of
the detonation. The MARLEN subscale was run at 1250 psia for 5 seconds
on methane/oxygen and showed two clean detonations in both the
microphone PSD (with peaks at 8.25 kHz and 16.665 kHz) and the
high-speed video. This is a big deal for flight applications because
launch and in-space propulsion both want high chamber pressure for
performance.

### Reduced injector pressure loss works

In traditional rocket engines, the injector pressure drop is kept high
to prevent combustion instabilities from coupling back into the
manifold. RDREs would normally inherit that same constraint. NASA
demonstrated wave activity from 700 to 1250 psia with injector pressure
losses ranging from only 100 to 415 psid. There were even conditions
where the pressure loss was so low that detonation waves were no longer
sustained, but no chug or other manifold instabilities appeared either.
This matters because every psi of injector pressure loss is a psi the
turbopump has to make up for.

### Fuel choice changes the behavior more than expected

Methane and kerosene gave similar wave counts for the same injector
design, with the wave speeds differing due to combustion kinetics and
mixing. Hydrogen was the outlier. At low pressures it gave a very
different wave count, and above about 250 psia chamber tap pressure it
just deflagrated. The interesting follow-up is that the measured
combustion efficiency and Isp for the deflagrating hydrogen case were
similar to a traditional steady-flow rocket. So an annular
hydrogen/oxygen rocket might still be useful in the near term for the
compactness, even without the detonation.

## My take

The single-wave damage result is the one I keep coming back to. In the
last paper I reviewed (Kickliter et al., 2025), the authors found that
single-wave operation is what the system naturally falls into when
initialized from a blast kernel, and you have to specifically engineer
the initial conditions to get a multi-wave mode to take hold. If you
combine that with what Teasley shows here, the picture is that the
default behavior of an RDRE both in simulation and in hardware is
single-wave, and that default is exactly the mode that breaks the
engine. So the question of how to reliably trigger and sustain
multi-wave operation is not just an academic curiosity. It is on the
critical path for flight hardware.

The other thing I want to point out is the manufacturing emphasis. A
lot of the paper is about which alloys can survive the heat fluxes,
which manufacturing methods can produce the geometry, and what post
processing is needed. The takeaway for me is that RDRE development is
not bottlenecked by the combustion physics anymore. It is bottlenecked
by being able to actually build the hardware. L-PBF GRCop-42 with
chemical mechanical polishing of the coolant channels comes up
repeatedly, and the paper basically says no other process will work at
these conditions. That is a strong statement and it lines up with what
I have seen in my own hands-on machining work at GTRX and GTRI, where
the difference between a design that works and one that does not often
comes down to whether the part can actually be made to spec.

One thing I would have liked to see more of is data on the hydrogen
deflagration cases. The paper mentions that combustion efficiency and
Isp were similar to traditional steady-flow rockets, but does not give
the actual numbers. If a deflagrating annular hydrogen/oxygen rocket is
being considered as a near-term option, then I think it would be worth
sharing the comparison directly so that the trade between compactness
and performance is clear.

## What I would want to see next

The thing I am most curious about is what the actual mechanism is
behind hydrogen's preference to deflagrate at higher pressures. The
paper says it is likely a design problem with injector or annulus
geometry promoting deflagrative losses, but it does not commit to
which. If it is geometry-driven, then the right injector or annulus
gap width should be able to bring the detonation back. If it is more
fundamental, related to the chemical timescales of hydrogen at high
pressure, then it is harder to fix with hardware changes alone. This
ties back to the $Da = \tau_{flow} / \tau_{chem}$ framing from the
Kickliter paper — hydrogen at high pressure may simply put the system
in a different region of that parameter space than methane or kerosene
do.

## References

1. Teasley, T., "NASA's Rotating Detonation Rocket Engine Development,"
   *AAS Conference*, 2025.
2. Kickliter, T. M., Young, E., Acharya, V., and Lieuwen, T. C.,
   "Asymptotic and Transient Dynamics of Rotating Detonation Engines,"
   *AIAA SciTech 2025 Forum*, AIAA Paper 2025-0399.
   [DOI](https://doi.org/10.2514/6.2025-0399)
3. Petty, D., Teasley, T., Hernandez-McCloskey, J., and Goldman, A.,
   "Parameterized Study of Heat Load Trends in a Subscale Rotating
   Detonation Rocket Engine," *AIAA SciTech 2025 Forum*, Jan. 2025.
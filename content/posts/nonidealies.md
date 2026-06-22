---
title: "Two CFD Projects: Running Fluent and Writing My Own Solver"
date: 2026-06-22
draft: false
math: true
tags: ["CFD", "compressible-flow", "nozzle", "finite-volume", "project"]
summary: "Two projects from this past year sat at opposite ends of how you actually do CFD — one was running a commercial solver on a rocket nozzle, the other was coding a finite-volume Roe solver from scratch. Doing them back to back changed how I think about what these tools are actually doing."
---

## Why I wanted to write this up

Two of the bigger projects I worked on this past year were both
computational fluid dynamics, but they sat at completely opposite ends of
how you actually do CFD. The first one, for AE 4040, was about running a
commercial solver — ANSYS Fluent — on a converging-diverging nozzle and
trying to pull real flow physics out of it. The second, for AE 6042, was
about writing my own finite-volume solver from scratch and watching it
break in ways the commercial tool would never let me see.

Doing them in that order ended up being the useful part. When you only run
Fluent, the solver is a black box that hands you pretty contours. When you
write the solver yourself, you find out exactly how much work is going on
under the hood to keep a supersonic flow from blowing up. I wanted to put
both in one place because together they say more about what I learned than
either one does on its own. Rocket propulsion is the area I want to work
in, and almost everything in that field eventually comes back to
compressible flow, so getting comfortable on both sides of the tool felt
worth the time.

## Project 1: A converging-diverging nozzle in Fluent

### Why a nozzle, and why the Raptor expansion ratio

I picked a nozzle because that is the geometry I keep running into in
propulsion, and I wanted the problem to connect to something real instead
of an abstract textbook duct. So I designed the nozzle around the sea-level
SpaceX Raptor expansion ratio of 34.34:1. I did not copy the actual Raptor
dimensions, just that area ratio, because the isentropic flow features
depend on the area ratio and not the absolute size. That meant I could
always scale the thing up later to true dimensions and the flow physics
would still hold.

I ran four back pressure conditions off a fixed stagnation inlet of
$P_0 = 300$ kPa and $T_0 = 300$ K. Two of them were meant to put a shock
inside the nozzle, and two were meant to give a clean supersonic expansion
with no shock at all.

To figure out which back pressures actually do that, I worked the
area–Mach relation with $A_e/A_t = 34.34$ and $\gamma = 1.4$:

$$\frac{A}{A^*} = \frac{1}{M}\left[\frac{2}{\gamma+1}\left(1 + \frac{\gamma-1}{2}M^2\right)\right]^{\frac{\gamma+1}{2(\gamma-1)}}$$

That gives a design exit Mach number of about $M_e \approx 5.40$, a
perfectly expanded pressure of about $0.357$ kPa, and a back pressure limit
of about $12.13$ kPa to hold a shock right at the exit. With the subsonic
branch worked out the same way, the window for a shock sitting inside the
nozzle came out to roughly $12.13 \text{ kPa} < P_b < 299.94 \text{ kPa}$.
That is why I chose 100 kPa and 50 kPa for the shock cases — both land
inside that window.

For the shock-free cases I just needed something below 12.13 kPa. I
originally picked 0.3 and 0.2 kPa, but those would not converge for me no
matter what I tried, so I bumped them up to 0.6 and 0.8 kPa, which still
sit in the supersonic range and actually ran. That kind of "the physics
says one thing but the solver says another" tradeoff showed up a lot in
this project.

### The mesh and solver setup

I built a 2D axisymmetric structured mesh in Pointwise. The part I spent
the most time on was the wall spacing. I wanted a $y^+$ around 10 so I
could actually resolve the boundary layer on the nozzle wall, and I refined
the mesh toward the throat since that is where the shock forms and where I
wanted the most resolution. Going much finer than that overpopulated the
near-wall cells and wrecked my aspect ratios, so 10 was the compromise I
landed on. The final mesh was about 29,500 cells, with a max aspect ratio
up around 8,500, which is high but acceptable for the thin near-wall cells.

In Fluent I ran a steady density-based solver since this is high-speed
compressible flow, which meant turning on the energy equation, and I used
the SST $k$–$\omega$ turbulence model. Boundary conditions were an axis for
the axisymmetric revolve, a pressure inlet, a pressure outlet, and a
no-slip stationary wall. I set the operating pressure to zero so my gauge
values were just the conditions I picked.

### What the shock cases showed

The 100 kPa case was the one I learned the most from. The flow chokes at
the throat, expands to about Mach 2.2 in the diverging section, and then
hits a shock a little downstream of the throat. What I did not expect was
how clearly the shock showed up as a lambda shock in the density gradient
plot. That structure comes from the boundary layer — the flow at the wall
is dragging from friction while the core is not, so the shock foot splays
out into that lambda shape near the surface. Right behind it the boundary
layer separates from the adverse pressure gradient and you get a
recirculation zone, which I could see as dark low-Mach pockets along the
wall and confirmed with the streamlines breaking up where the vorticity
spiked.

On the centerline, the static pressure dropped to about 28 kPa just ahead
of the shock and then jumped to around 160 kPa across it, while the Mach
number snapped from about 2.2 down to 0.72. The boundary-layer velocity
profiles told the same story from a different angle — at the throat and
before the shock the profiles go cleanly to zero at the wall, but after the
shock the near-wall velocity actually goes negative before coming back to
zero, which is the recirculation showing up in the data.

Dropping the back pressure to 50 kPa pushed the shock further down the
nozzle and made it stronger. The flow had more length to keep accelerating
before the shock, so the pre-shock pressure dropped further (down to about
10 kPa) and the jump across the shock was bigger. Past the shock I could
see shock diamonds marching toward the outlet from the oblique shocks
reflecting off each side, which was a nice bonus to catch.

### Where the simulation and the theory disagreed

For the two shock-free cases the flow expanded smoothly with no jumps, just
small oscillations, which is what I wanted. The interesting part was
comparing the simulated exit Mach against the isentropic value. The
area–Mach relation says the exit should be about Mach 5.4, but the
simulation came out closer to 4.6. That gap is the whole point of running
CFD instead of trusting the isentropic relations — once you add viscosity
and a real boundary layer, you lose some of that ideal expansion, and the
theory does not capture it. Seeing the number actually come out lower made
that lesson concrete instead of something I just read in a slide.

## Project 2: Writing a Roe solver from scratch

### Going from running a solver to being the solver

The second project flipped everything. Instead of setting up a case in
Fluent and letting it solve, I had to write a 2D finite-volume Euler solver
myself, in MATLAB, for supersonic flow over a diamond-shaped airfoil at
zero angle of attack. I used the Roe Flux Difference Splitting scheme, and
I had to run three versions of increasing accuracy: first-order, a
second-order fully upwind MUSCL formulation, and a third-order QUICK scheme
with a minmod limiter.

The core of the whole thing is the Roe numerical flux at each face:

$$\hat{F} = \frac{1}{2}\left(F_L + F_R\right) - \frac{1}{2}\sum_{k=1}^{4} \alpha_k\,|\tilde{\lambda}_k|\,\tilde{r}_k$$

The first term is just the average of the left and right fluxes, and the
second term is the dissipation that the Roe averaging adds in based on the
wave strengths $\alpha_k$, the eigenvalues $\tilde{\lambda}_k$, and the
right eigenvectors $\tilde{r}_k$. Writing this out by hand — computing the
Roe-averaged density, velocities, enthalpy, and speed of sound, building
the eigenvalue and eigenvector matrices, and getting the wave strengths
right — is where I actually understood what a solver is doing every time it
takes a step. In Fluent that is one dropdown. Here it was a couple hundred
lines and a lot of indexing I had to get exactly right.

I used local time stepping with a CFL of 0.4 and tracked both the $L_2$ and
$L_\infty$ residual norms, stopping when the flow was converged or after
50,000 iterations.

### The part that did not work

I want to be honest about this because it was the most useful part. The
first-order case converged on both grids, and the second-order case
converged on the finer grid. But the third-order case with the minmod
limiter never converged for me — the residuals just never came down past
the tolerance. My approach was to take the MUSCL reconstruction and fold
the limiter function into it through the gradient ratio, but something in
that implementation kept the solution from settling. I also could not get
the first two cases all the way down to machine accuracy and had to relax
the tolerance a bit to call them converged.

A year ago I think a non-converging case would have just felt like a
failure. After doing the nozzle project first, it read differently — it is
the same convergence struggle, except now I own every line that could be
causing it. The limiter, the reconstruction, the boundary halo cells, the
time step. There is nobody else's solver to blame, which is exactly why it
was worth doing.

### What the converging cases showed

Comparing the two grids on the first-order case, the finer grid gave
slightly different values and a thinner shock, which lines up with less
numerical dissipation on the tighter mesh. Going up to second order
sharpened things a lot more — the shock came out straighter and more
clearly angled, and I could actually see the expansion fan off the back of
the diamond, where the first-order solution just smeared everything into
curves. When I checked the values against weak oblique shock theory, the
higher-order case landed noticeably closer to the theoretical numbers than
the first-order one did. The tradeoff was runtime: more accuracy meant a
longer solve, which is its own lesson about why people make the choices
they do in production CFD.

## What I took away from doing both

The nozzle project taught me how to drive a real solver — meshing for $y^+$,
picking a turbulence model, reading shock structure and separation out of
contours, and seeing where viscosity pulls the answer away from the
isentropic ideal. The Roe solver taught me what that driver is sitting on
top of, all the way down to the dissipation term that keeps a supersonic
shock stable.

The thing I keep coming back to is that the two halves cover for each
other. After writing the Roe scheme, I trust Fluent more, because I have
some sense of what it is protecting me from. And after fighting Fluent's
convergence on the nozzle, I had a lot more patience for my own solver not
converging, because I knew that part was always going to be hard. For
propulsion work, where compressible flow is everywhere and the answers
matter, being comfortable on both sides of the tool feels like the right
foundation to be building.

## Tools used

ANSYS Fluent and Pointwise for the nozzle simulation and mesh, ParaView for
post-processing and plotting, and MATLAB for the hand-coded Roe finite-volume
solver.
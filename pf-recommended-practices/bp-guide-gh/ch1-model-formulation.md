# Model Formulation

Nana Ofori-Opuku, Jim Warren, Pierre-Clement Simon

* Review the literature
  * Start with a careful lit review and see what others have done.
  * Take the time to understand the derivations from the literature.
  * Consider good examples from the literature (Can we list these on PFHub?)

## General Considerations on the formulation of phase field models

Phase field models are, quite generally, extensions of classical
non-equilibrium thermodynamics. There are quite a few treatments of this in the
literature.  Here we will follow formulations similar to those developed by
Sekerka and Bi in "Interfaces for the Twenty-First Century."
{cite}`bi_phase-field_1998` In that spirit, we will start with a very general
formulation, including hydrodynamics, and then simplify the problem.  For those
interested in starting with a simple model, you can skip over much of the
initial formulation and jump to the section on the further reduction of the
problem.

In thermodynamics one usually starts from consideration of the internal energy
$E$, where $E=E(S,V,M_i)$, and $S$ is the entropy, $V$ is the volume and $M_i$
is the mass of species $i$. If we decide we don't want to consider elasticity
we can divide through by the volume and compute the bulk energy density
$e=e(s,\rho_i)$, with $\rho_i=M_i/V$, and similarly $s=S/V$. It is then usually
postulated that in addition to the thermodynamic variables there are any number
of _phase fields_ $\phi_j$ and the internal energy density is written

$$e=e(s,\rho_i,\phi_j).$$

Here we are going to consider a single phase field $\phi$ that distinguishes
between two phases (could be liquid-solid, could be solid-gas, etc.) and
postulate that the total entropy of a system of volume $V$ can be written as

$$S = \int s^{NC} dV,$$

where

$$s^{NC}=s-\frac{\epsilon^2}{2}|\nabla\phi|^2-\sum_i \frac{\alpha_i^2}{2}|\nabla\rho_i|^2.$$

Here the superscript $NC$ is meant to indicate that this is a non-classical
definition of the entropy density $s$. We have chosen the simplest possible
forms of the non-classical extensions (that is, simple square-gradient
penalties). The entropy density is itself a function, of course,

$$s=s(e,\rho_i,\phi).$$

To close the deal we now need to work our way through the various
continuum-versions of the laws of nature.

### Evolution equations

In order to write down evolution equations we need to resort to the known laws
of physics. For continuum theories, we really have only a few ideas to fall
back on

* Conservation of mass
* Conservation of energy (First Law of Thermodynamics)
* Conservation of momentum and maybe angular momentum if absolutely necessary
* The second law of thermodynamics (entropy increases, free energy decreases...)

A complete discussion of how to use the above rules in this context is outside
the scope of this best-practice guide, but we offer a highly abbreviated
discussion of the ''flavor,'' following the ideas of irreversible
thermodynamics (we largely are following works like deGroot and Mazur
{cite}`groot_non-equilibrium_2013`, although the continuum mechanics community
may be more comfortable with Noll, Coleman, and Truesdale
{cite}`malvern_introduction_1969`), as well as the aforementioned work by
Sekerka and Bi {cite}`bi_phase-field_1998`.

### Mass

The first idea is that we need to specify a flux of particles

$${\bf J}_i=\rho_i \left({\bf v}_i-{\bf v}\right),$$

where ${\bf v}_i$ is the velocity of the particles of species $i$.
In general we want to work in a center-of-mass frame

$${\bf v}=\frac{\sum_i \rho_i {\bf v}_i}{\sum_i \rho_i}.$$

Notice that this implies (by construction) that $\sum_i {\bf J}_i=0$.

The law of conservation of mass can be simply written as

$$\frac{\partial \rho_i}{\partial t}+\nabla\cdot(\rho_i {\bf v}_i)=0.$$

Then we use the definition of ${\bf J}_i$ to rewrite that as

$$\frac{\partial \rho_i}{\partial t}+ \nabla\cdot\left({\bf J}_i+\rho_i{\bf v}\right)=0.$$

Note that if we sum this equation over $i$, we get the continuity equation

$$\frac{\partial \rho}{\partial t}+ \nabla\cdot\left(\rho{\bf v}\right)=0.$$

At this point it is common to introduce the notation

$$\frac{D}{Dt}=\frac{\partial}{\partial t}+{\bf v}\cdot\nabla,$$

the "Lagrangian derivative" or "material derivative". Then we can write the
mass conservation equation as

$$\frac{D \rho_i}{Dt}+ \nabla\cdot{\bf J}_i+\rho_i\nabla\cdot{\bf v}=0.$$

If the system in incompressible (a common assumption, although we'd have to
abandon that if we want to consider gasses), then $\nabla\cdot{\bf v}=0$ and we
can write

$$\frac{D \rho_i}{Dt}+ \nabla\cdot{\bf J}_i=0.$$

### Momentum

The law of conservation of momentum (Newton's second Law) for a system without
body forces (like gravity) can be written

$$\frac{\partial\rho {\bf v}}{\partial t}+\nabla\cdot\left( \rho {\bf
v}\otimes{\bf v}-\sigma\right)=0,$$

where $\sigma$ is the stress tensor.  Using continuity and incompressibility as
assumptions we get

$$\rho\frac{D{\bf v}}{Dt}=\nabla\cdot\sigma.$$

### Energy

Our final conservation expression is for energy.  Just like the equation for
momentum and mass, we start with a continuity form, and assume a flux of
internal energy ${\bf J}_e$, and distinguish between the total energy density
$e_T$, the internal energy density $e$ and the kinetic energy density
$e_k=e_T-e= \frac{1}{2}\rho v^2$ and write

$$\frac{\partial e_T}{\partial t}+\nabla\cdot\left(e_T{\bf v}+{\bf J}_e\right)=0.$$

We can then use the momentum equation as well as the mass continuity equation
(along with incompressibility) to arrive at

$$\frac{D e}{D t}+\nabla\cdot{\bf J}_e=\nabla{\bf v}:\sigma.$$

### First Law of Thermodynamics

We now write down the laws of thermodynamics for the densities.

$$e=Ts-p+\sum_i\rho_i\mu_i,$$

where $p$ is the hydrostatic pressure,

$$p=-\frac{1}{3}{\mathrm{Tr}}\sigma,$$

and $\mu_i$ and $T$ are defined as derivatives of $e$ through the expression

$$de=Tds+\sum_i\mu_i d\rho_i+\frac{\partial e}{\partial\phi}d\phi.$$

Of course, given where we're headed, let's rewrite this as

$$ds=\frac{1}{T}de-\sum_i\frac{\mu_i}{T}d\rho_i+\frac{\partial s}{\partial \phi}d\phi.$$

### Second Law

We have chosen the entropy to "extend" non-classically, as it is the star actor
in the second law of thermodynamics, so we can write the entropy production
(which must be positive) as

$$S^{\mathrm{prod}}=\frac{dS}{dt}+\int {\bf J}_s\cdot d{\bf A}\ge0,$$

where ${\bf J}_s$ is the flux of entropy.  We can compute (for an incompressible system)

$$S^{\mathrm{prod}}=\int \left[\sum_i -\left(\frac{\mu_i}{T}\right)^{NC}\frac{D\rho_i}{Dt}+S_\phi\frac{D\phi}{Dt}+\frac{1}{T}\frac{De}{Dt}\right]dV+\int \left[{\bf J}_s-\sum_i \alpha_i^2\nabla\rho_i\frac{D\rho_i}{Dt}-\epsilon^2\nabla\phi\frac{D\phi}{Dt}\right]\cdot d{\bf A},$$

where we have introduced

$$-\left(\frac{\mu_i}{T}\right)^{NC}=\frac{\partial s}{\partial \rho_i}+\alpha_i\nabla^2\rho_i,$$

and

$$S_\phi=\frac{\partial s}{\partial\phi}+\epsilon^2\nabla^2\phi.$$

At this point we can insert the equations of motion we got from our
conservation laws and do _a lot_ of algebra, which we leave as an amusing
exercise for the reader.  We eventually find that

$$S^{\mathrm{prod}}=\int \left[-\sum_i\nabla\left(\frac{\mu_i}{T}\right)^{NC}\cdot{\bf J_i}+S_\phi\frac{D\phi}{Dt}+\nabla \frac{1}{T}\cdot{\bf J}_e+\frac{1}{T} {\bf Y}:\nabla {\bf v}\right]dV,$$

and

$${\bf J}_s=s^{NC}{\bf v} + \frac{1}{T}{\bf J}_e-\sum_i \left(\frac{\mu_i}{T}\right)^{NC}{\bf J}_i+\sum_i \alpha_i^2\frac{D\rho_i}{Dt}\nabla\rho_i+\epsilon^2\frac{D\phi}{Dt}\nabla\phi.$$

It is worth noting that this form for ${\bf J}_s$ eliminates the explicit
surface terms from the entropy production. We have also introduced the tensor
${\bf Y}$ which is rather a complicated beast (it came from all those
integrations by parts which we have skipped in this presentation), and has the
form

$$\frac{{\bf Y}}{T}=\frac{\sigma}{T}+\sum \alpha_i^2\nabla\rho_i\otimes\nabla\rho_i+\epsilon^2\nabla\phi\otimes\nabla\phi+\left(\frac{p}{T}-\sum_i \alpha_i^2\rho_i\nabla^2\rho_i-\sum_i\frac{\alpha_i^2}{2}|\nabla\rho_i|^2-\frac{\epsilon^2}{2}|\nabla\phi|^2\right){\bf I},$$

where $\bf I$ is the identity tensor.

### Constitutive Equations and Evolution Equations

After all this work we can now find constitutive equations.  Inspection of the
final expression for $S^{\mathrm{prod}}$, suggests a that we can guarantee it
be positive definite (which is required from the second law) by assuming

$${\bf J}_i= -M_i\nabla\left(\frac{\mu_i}{T}\right)^{NC},$$

$${\bf J}_e=M_T\nabla\frac{1}{T},$$

$${\bf Y} = \eta \left(\nabla{\bf v}+(\nabla{\bf v})^T\right),$$

$$\frac{D\phi}{Dt}=M_\phi S_\phi,$$

where we have introduced mobilities and the viscosity $\eta$. It should be
emphasized that we could have assumed all sorts of "cross-couplings" between
the various fluxes here, but instead we have made the simplest
assumptions. Inserting these into our equations of motion, we (at last!) arrive
at general evolution equations

$$\frac{D e}{D t}=-M_T\nabla^2\frac{1}{T}+\nabla{\bf v}:\sigma,$$

$$\frac{D \rho_i}{D t}=\nabla\cdot M_i\nabla\left(-\frac{\partial s}{\partial \rho_i}-\alpha_i^2 \nabla^2\rho_i\right)$$

$$\frac{D\phi}{Dt}=M_\phi\left(\frac{\partial s}{\partial\phi}+\epsilon^2\nabla^2\phi\right).$$

$$\frac{\sigma}{T}=\frac{\eta}{T}\left(\nabla{\bf v}+(\nabla{\bf v})^T\right)-\sum_i \alpha_i^2\nabla\rho_i\otimes\nabla\rho_i+\epsilon^2\nabla\phi\otimes\nabla\phi-\left(\frac{p}{T}-\sum \alpha_i^2\rho_i\nabla^2\rho_i-\sum\frac{\alpha_i^2}{2}|\nabla\rho_i|^2-\frac{\epsilon^2}{2}|\nabla\phi|^2\right){\bf I},$$

with

$$\rho\frac{D{\bf v}}{Dt}=\nabla\cdot\sigma.$$

### Further reduction of the problem

We have presented the derivation of equations of evolution for an
incompressible, two-phase, multicomponent system.  Most phase field treatments
ignore the velocity terms, but for pedagogical purposes we have retained all of
the terms that arise from flow (for an incompressible system). These terms
should not just be tossed away without careful consideration of the system of
interest! Nonetheless, now that we have taken care to show how these terms
arise, and also, for careful readers, how to extend this approach to multiple
phases and additional gradient corrections, we will proceed with some more
simplifications for a less complex system.  For those who are interested in
solid state systems that can creep, the work of Mishin, Warren, Sekerka, and
Boettinger (2013) {cite}`mishin_irreversible_2013` extends this framework.
Here we eliminate the ${\bf v}$ equations by fiat, assuming that only diffusion
controls the evolution of the system, which is often reasonable in microgravity
situations.  We can also go further, and consider an isothermal system. Then we
have

$$\frac{\partial \rho_i}{\partial t}=\nabla\cdot M_i\nabla\left(-\frac{\partial s}{\partial \rho_i}-\alpha^2_i\nabla^2\rho_i\right),$$

$$\frac{\partial\phi}{\partial t}=M_\phi\left(\frac{\partial s}{\partial\phi}+\epsilon^2\nabla^2\phi\right).$$

This system of equations should be adequate to describe a diffusion-controlled
multicomponent, isothermal, two-phase system. One could, of course, retain
thermal diffusion, add more phases, retain the velocity terms and more!

## Clearly define the equations, parameters, initial conditions, and boundar conditions

Now that we have specified that we wish to consider, a diffusion-controlled
multicomponent, isothermal, two-phase system. We still need to decide on the
specifics of the system, and, in particular, the thermodynamic state functions
that detail how the energy or entropy vary.  All of the variables need to be
clearly defined, and ideally should be either parameters that can be traced to
thermodynamic variables or other physical constraints.  As noted, we wish to
consider an isothermal system, and so we can use a Legendre transformation to
replace $s$ in the above expressions with the Helmholtz free energy $f=e-Ts$.

$$\frac{\partial \rho_i}{\partial t}=\nabla\cdot M_i\nabla\left(\frac{1}{T}\frac{\partial f}{\partial \rho_i}-\alpha^2_i\nabla^2\rho_i\right)$$

$$\frac{\partial\phi}{\partial t}=M_\phi\left(-\frac{1}{T}\frac{\partial f}{\partial\phi}+\epsilon^2\nabla^2\phi\right).$$

Additionally, we will now only consider only a two component systems, so we
only have species $\rho_1$ and $\rho_2$. Note that the total density
$\rho=\rho_1+\rho_2$ is a constant (for an incompressible system).  We could
formulate the problem in terms of these variables or we can move to a
concentration $c_1=\rho_1/(\rho_1+\rho_2)$.  Clearly $c_1+c_2=1$, so we only
have one independent variable. In this case we can pick $c_1$ or $c_2$ as the
independent variable and label it $c$.  As the reader can see, these types of
exercises are highly non-trivial, require substantial care, and need to be done
with rigor to understand the host of approximations and assumptions that can be
made (at a minimum here we have ignored elasticity, different molar volumes for
the species, and other issues are external fields, and there will be more to
come!). We move to a concentration picture and _redefine_ the mobilities,
$\alpha$, and $\epsilon$ in such a way that the following expressions hold:

$$\frac{\partial c}{\partial t}=\nabla\cdot M_i\nabla\left(\frac{\partial f}{\partial c}-\alpha^2\nabla^2c\right)$$

$$\frac{\partial\phi}{\partial t}=M_\phi\left(\epsilon^2\nabla^2\phi-\frac{\partial f}{\partial\phi}\right).$$

### State function

We now need to specify $f(c,T,\phi)$.  Note that we have retained the $T$
dependence here, even though we are considering an isothermal
system. Isothermal does not mean temperature independent! Additionally, in the
expressions just above we "absorbed" $T$ into some of the coefficients.  These
parameters can depend on $T$, but often the dependencies are unknown or should
only be included if the phenomena under consideration demand such an
inclusion. In the interest of pedagogy, we recapitulate one approach to
modeling a liquid-solid binary alloy, although the details are less important
that understanding that a specific choice of state function has to come from
_somewhere_. Following the treatment in the Annual Reviews of Materials
Research (2001) by Boettinger, Warren, Beckerman and Karma
{cite}`boettinger_phase-field_2002` we note that the free energy can be
determined through a multi-step process where the two components are called $A$
and $B$ respectively:

#### Step 1

Start with the ordinary free energy of the pure components as liquid and solid
phases, $f_L^A(T)$, $f_L^B(T)$, $f_S^A(T)$, $f_S^B(T)$. They are functions of
temperature only.

#### Step 2

Form a function that represents both liquid and solid for pure A

$$f^A(\phi,T)=(1-p(\phi))f^A_S+p(\phi)f^A_L+W^Ag(\phi).$$

This function combines the free energies of the liquid and solid with the
interpolating function $p(\phi)$ and adds an energy hump, $W^Ag(\phi)$, between
them. A similar expression can be obtained for component B. Note that $g(\phi)$
has minima at $\phi=0$ and $\phi=1$ with a maximum at $\phi=0.5$ while
$p(0)=0$, $p(1)=1$ and $p'=0$ when $\phi=0$ or $\phi=1$.  A popular choice for
these functions is $p(\phi)=\phi^3(10-15\phi+6\phi^2)$ and
$g(\phi)=\\phi^2(1-\phi)^2$.  Note also the $\partial p/\partial\phi=30g(\phi)$
by construction.

#### Step 3

Form the function, $f(\phi,c,T)$, that represents (for example) a regular
solution of A and B,

$$f(\phi,c,T)=(1-c)f^A(\phi)+cf^B(\phi) +RT((1-c)\ln(1-c)+c\ln c)+c(1-c)\left[\Omega_S(1-p(\phi))+\Omega_L p(\phi)\right],$$

where $\Omega_L$ and $\Omega_S$ are the regular solution parameters of the
liquid and solid that again are combined with the interpolating function
$p(\phi)$. The gas constant R needs to be chosen with the correct normalization
so that RT has units of energy per unit volume.

With this specific choice of a regular solution free energy we at last have
fully specified a system and its evolution!

## Clearly define and pose your physical problem before solving

Regardless of the explicit functional choices for the state function, the most
important point made above is that the phase field method requires an explicit
choice for this function.  Once we have selected it, the equations derived
above detail the evolution of the phase, composition, temperature, etc.  Here
we chose only to consider phase and composition, with temperature as a
parameter. Having fully specified the state function, the next question is
really about practical issues around a specific simulation. What are the
boundary conditions? What are the relevant length scales.  Here is a list of
issues:

### Complete dimensional analysis and understanding relevant scales; state the reference frame

There are any number of dimensionless parameters that can be found in these
models.  Understanding their relative role is beyond the scope of this best
practice guide, but constructing these numbers is straightforward and
essential. Usually before attempting to solve the equations, it is useful to
"scale" the equations, thereby reducing the total number of parameter in the
problem. In general we can always rescale space and time to eliminate two
parameters. For example, a natural unit for length $\ell$ might be defined in
terms of a scale of the free energy density $\bar f$

$$\ell^2=\frac{\epsilon^2}{\bar f}.$$

Using this definition of $\ell$ we can then get a time scale $\tau$ as

$$\tau=\frac{1}{M_\phi\bar f}.$$

Using these units will eliminate two parameters that need to explored when
seeking solutions.

### Plot your state functions before implementing them in code

While we derived a specific choice of the free energy above, regardless of
where you got the free energy (or other state function) it is wise to plot the
function, as this will give you a quick check that your state function is well
posed, has minima where you expect them to be.

### Consider the impact of interpolation functions, barrier functions, etc.

We chose specific forms for our interpolation function $p$ and the barrier
function $g$. These were not arbitrary choices, but nonetheless, they are
hardly the only choices we could have made, just relatively popular. All such
choices can result in simulation regions where "surprises" can occur, often due
to insufficient spatiotemporal resolution.

### Consider every boundary term and make sure you aren’t eliminating important terms. Check the fluxes

When implementing the boundary conditions, extreme care must be taken.  One
cannot just "zero-out" boundary terms, as this may break some requirement about
mass conservation or other physical constraint.

* (Show examples on PFHub, possibly benchmark problems?)
* Be mindful of your assumptions and approximations

Having clearly defined the mathematical framework underlying the problem, it
becomes important, prior to progressing further, to contemplate and thoroughly
grasp the various assumptions and approximations inherent within the
formulation. Often, the exacting mathematical underpinnings from which the
model has arisen may have presented challenges of being intractable in their
complexity, or alternatively, they might have been of an impractical nature or
even lacking contact with empirical observations.

The considerations surrounding assumptions and approximations are distributed
throughout various places throughout the model's formulation. Foremost, are the
descriptors of thermodynamic energy density, which provide the basis for our
understanding of energy relationships, as well as the delineations of boundary
conditions and the specifications of initial conditions. Notably, the latter
two warrant attention since, in the absence of these conditions, the partial
differential equations that determine the temporal evolution of our problem
could only hypothetically yield an infinite set of solutions.

* If possible, run a small test problem with and without approximations and to
  quantify their impact
* Consider how approximations change in different dimensions and symmetries
* If possible, avoid assuming symmetry to reduce the complexity of your problem

## Carefully consider your length and time scales

1. **Identify the Smallest Feature Size of Interest in Your Problem:** The
   smallest feature size refers to the smallest spatial scale that needs to be
   accurately resolved in your simulation. This could be, for example, the
   width of an interface between two distinct materials or the size of a fine
   structure. Properly resolving the smallest features is essential for
   capturing detailed interactions and behaviors in your system.

   **Additional Item 1: Generally the smallest feature is the interface width
   between the features that need to be resolved.** The interface width often
   represents the smallest detail that demands attention. Neglecting this width
   may lead to inaccurate results and incomplete understanding of the system's
   behavior.

2. **Identify the Smallest Time Scale of Interest in Your Problem:** The
   smallest time scale corresponds to the fastest processes occurring in your
   system. It's important to accurately capture these time scales to ensure
   that rapid events are not overlooked or misrepresented.

   **Additional Item 2: The time steps you consider shall be small enough to
   allow capturing the kinetics of the process you want to model.** Choosing
   appropriate time steps is crucial to accurately capture dynamic
   processes. Smaller time steps are necessary to capture fast kinetics and
   prevent underestimation or distortion of time-dependent phenomena.

3. **Identify the Smallest Length Scales of Interest for All Physics and
   Consider How They Interact:** This involves analyzing the smallest spatial
   scales relevant to each physical process in your simulation. Understanding
   how these different length scales interact is essential for capturing
   multi-scale effects accurately.

   **Additional Item 3: Depending on the numerical method used for solving the
   equations, you may need 5-10 mesh elements to resolve the interface.** The
   mesh resolution, i.e., the number of mesh elements used to discretize the
   domain, plays a crucial role in capturing interfaces accurately. Higher mesh
   resolution is required to resolve fine interfaces properly.

4. **The Sample Size You Consider Will Be Relevant to Your Feature Size:** The
   size of the simulation domain should be appropriately chosen based on the
   scale of the features you are interested in. Having a domain that is much
   larger than necessary can lead to unnecessary computational costs, while
   having a domain that is too small might exclude important interactions.

5. **It Is Recommended to Formulate Your Problem in a Non-Dimensional Form If
   Possible:** Non-dimensionalization involves scaling variables in your
   equations to remove physical units. This can help simplify the equations,
   reduce the number of parameters, and make the problem more amenable to
   analysis and comparison.

6. **If You Consider an Initial Condition Far from the Expected Steady State
   Solution, You May Need to Select a Significantly Smaller Time Step:** When
   the initial condition differs significantly from the expected steady state,
   smaller time steps may be necessary to accurately capture the transient
   behavior and prevent numerical instability.

7. **Be Mindful of Initial Conditions You Select:** Poorly chosen initial
   conditions, especially for interfaces or abrupt changes, can lead to
   numerical artifacts, excessive computational demands, and inaccurate
   results. It's important to select physically meaningful initial conditions
   that smoothly transition into the desired system state.

   **Additional Item 7: They may also result in numerical artifacts that lead
   to artificial stabilization of unstable phases.** Inaccurate initial
   conditions can stabilize phases that would otherwise be unstable, leading
   to unrealistic outcomes in your simulation.

8. **Using Automatic Time Stepping Integrator Algorithms Can Benefit PF
   Simulations:** Automatic time stepping algorithms adjust the time step size
   during simulation based on the system's dynamics. This can improve
   simulation efficiency by using larger time steps during stable periods and
   smaller steps during dynamic events.

## Summary

In summary, these considerations highlight the importance of careful parameter
selection, proper resolution of length and time scales, and accurate initial
conditions when performing numerical simulations. Addressing these aspects
helps ensure the reliability and validity of your simulation results in various
scientific and engineering applications.

* Non-dimensionalization
  * It gets MUCH harder with coupled physics, so be careful

## References

```{bibliography}
```

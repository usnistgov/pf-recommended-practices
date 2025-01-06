# Numerical Implementation

When writing your own phase-field simulation code or when choosing among
existing codes, the numerical implementation of the model is an important
consideration. This section of Phase-Field Recommended Practices Guide is split
into two subsections -- one focused on choosing appropriate numerical methods
and one focused on choosing whether to code your own solvers or use existing
libraries (and how to choose between existing libraries). Although these
subsections are presented sequentially, these choices are more iterative than
sequential. For example, you may start out deciding that a certain set of
numerical methods are promising, but then switch to others after considering
the availability of libraries.

## Choosing an appropriate collection of numerical methods

There is no consensus on what the "best" numerical methods for phase-field
modeling are. In fact, given the diversity of the models and settings under the
phase field umbrella, it is unlikely that any single collection of methods
would be optimal for all phase-field modeling. Instead, a wide variety of
numerical methods are used in the literature. For most problems of interest
there are multiple numerical methods that can be used to obtain a solution to
the model equations (and perhaps some methods that _cannot_ obtain a
solution). Several factors to consider when choosing between numerical methods
are:

- General computational performance
- Appropriateness for the planned computing hardware (e.g., GPUs)
- Scalability, ease of implementation
- Availability of libraries

Each researcher may have their own unique balance of these factors that leads
them to choose a numerical approach.

These choices can be very important. The performance of different methods for a
given problem can vary by orders of magnitude. Likewise the complexity to
implement the methods can vary by orders of magnitude -- some solvers can be
written in a few tens of lines of code; some may take tens of thousands of
lines. The path-dependency of the choices are also variable. Transitioning from
a spectral code to a finite element code might necessitate an entire
rewrite. Transitioning from one implicit time integration method to another may
take a few hours.

A phase-field code involves a few different types of numerical
methods. Different choices for each type are possible, but not all combinations
are possible or advisable. In the end, to determine the numerical
implementation for your phase-field model you will need to decide on a
collection of numerical methods and approaches that are suitable for your
problem and work well together.

Before deciding on a strategy for a numerical implementation, we strongly
recommend that you put thought into your problem(s) of interest. Understanding
the governing equations, boundary conditions, initial conditions, coupled
physics, expected symmetries and other factors discussed in the [Model
Formulation section](https://github.com/usnistgov/pfhub/wiki/Model-Formulation)
will aid you in deciding on how to implement your model numerically.

### Start simple, then selectively move to more complex methods

In general, we recommend that you start with a simple method and then add
complexity selectively where justified. There are a few benefits to this
approach.
1. Simpler methods are usually easier to implement, decreasing the time from
   starting to running your first test simulation. This gives you an earlier
   opportunity to identify unexpected issues in your model formulation that you
   didn't catch before implementing it. Part of what makes a simpler method
   easier to implement is that a simpler method is often easier to debug and/or
   there are fewer ways for it to fail.
2. The simpler implementation can be a backup option if the development of a
   more complex option takes longer than expected. Even following the
   [recommended practices for software
   development](https://github.com/usnistgov/pfhub/wiki/Software-Development),
   code development times can be unpredictable. A suboptimal backup
   implementation might get you through your first milestones while you're
   debugging a more advanced method.
3. You may learn that the simpler option fulfills your needs. Complex methods
   aren't always better; the problem of interest might not need or justify
   investment in more complex methods.
4. Even if the simpler method doesn't fulfill your needs, it may illuminate
   where added complexity is worth investigating ("these small explicit time
   steps are bogging me down, I should try an implicit time integrator", "I'm
   wasting so many resources in this area where nothing happens, I should try
   adaptive meshing").
5. Solutions obtained with the simpler method can be used as reference
   solutions as part of your [code verification
   strategy](https://github.com/usnistgov/pfhub/wiki/Software-Development). Obtaining
   the same solution with two methods is no guarantee that both are correct,
   but it is an encouraging sign. And certainly, if two methods yield different
   solutions, the implementations require attention.

This all said, there may be some compelling reasons to start with more complex
methods. One such reason is if the method is already implemented in an
off-the-shelf package. Other numerical considerations could come into play: an
unpreconditioned Krylov solver is _simpler_ than a preconditioned one, but the
preconditioner stabilizes the solution and may be more forgiving for
non-optimized simulation parameters in an initial exploratory phase for a
project and may be _simpler to use_.

Additionally, we also recommend initially implementing a simpler form of your
model and the building up to the full model, particularly if you are new to the
toolchain that you're using. There are classes of issues that with your
numerical implementation that show up in even very simple calculations (MPI
communication issues, stencil issues, scaling issues, etc.), and it's best to
deal with those as soon as possible. Note: this approach works best when the
simplified model is a simplified version of the _PDE_, not simply a simplified
version of the _physics_. Sometimes models at different levels of complexity
have a very different structure. A few examples of simpler starting models are:

| Target Model | Simplified Variant |
| :---: | :---: |
| BM1 (Cahn-Hilliard) | Diffusion equation, then add the higher order term |
| BM2 (Coupled Cahn-Hilliard-Allen-Cahn) | Cahn-Hilliard by itself, add one Allen-Cahn equation, then two, then all of them|
| BM4 (Elastic Precipitate) | Cahn-Hilliard by itself, then add elasticity|

### Choosing a spatial discretization

Four main types of spatial discretizations are used for phase-field modeling:
finite difference, finite volume, finite element, and Fourier
pseudospectral. Each of these methods has been used for simulation results
uploaded to PFHub. Examples for each method are:

- finite difference: [Memphis](https://github.com/memphis-snl/memphis),
 [HiPerC](https://github.com/usnistgov/HiPerC)
- finite volume: [FiPy](https://github.com/usnistgov/fipy)
- finite element: [MOOSE](https://github.com/idaholab/moose),
 [PRISMS-PF](https://github.com/prisms-center/phaseField),
 [Fenics](https://github.com/FEniCS)
- Fourier pseudospectral: [MEUMAPPS](https://code.ornl.gov/meumapps/meumapps)

Here we give a brief overview of the strengths and weaknesses of each of these
spatial discretization approaches. Aiming for brevity over completeness, this
discussion skips over the considerable nuance in the capabilities of these
methods.

#### Finite difference

Finite difference discretizations are one of the most common spatial
discretizations for phase-field codes. The field data is stored on the vertices
of the mesh and derivatives are approximated using variable values at the point
of interest and/or neighboring points. See the [Wikipedia
page](https://en.wikipedia.org/wiki/Finite_difference_method) for a general
introduction. While many flavors of finite difference methods exist, most
finite difference phase-field codes use structured, uniform grids.

#### Finite volume

With finite volume methods, the field data is stored for each element in the
mesh (i.e., the "finite volume"), and changes to the field data are calculated
by summing the fluxes through the faces of the element. Therefore,
finite-volume methods are inherently conservative. Because the flux
calculations are done on the faces, finite volume methods can be used on
unstructured meshes. Also, as a result of the emphasis on fluxes, finite volume
methods solve the weak form of a PDE. See the [Wikipedia
page](https://en.wikipedia.org/wiki/Finite_volume_method) for a general
introduction.

#### Finite element

Finite element methods are the most complex of the common spatial
discretization methods for phase-field codes. In a finite element method, field
values are stored on pre-defined places of each element (the "nodes"). The
field variables in the element are continuously represented by a function of
pre-defined type (e.g., bilinear) that passes through the field values at each
node (the "shape function"). Derivatives are calculated through derivatives of
the shape function. The field variables are evolved using volumetric integrals
over the element, where the integrals are calculated using
[quadrature](https://en.wikipedia.org/wiki/Gaussian_quadrature). Therefore,
although field variables are stored at the nodes for the element, the evolution
expressions are calculated on "quadrature points", which often are not in the
same locations as the nodes. Due to the expressions being integral expressions,
finite element methods operate on the weak form of a PDE. See the [Wikipedia
page](https://en.wikipedia.org/wiki/Finite_element_method) for a general
introduction.

#### Fourier pseudospectral

Along with finite difference methods, Fourier pseudospectral methods are
historically a common spatial discretization for phase-field codes. Although
Fourier pseudospectral methods have similar theoretical underpinnings as finite
element methods, in practice, implementing a Fourier pseudospectral code is
very different from the other methods listed here. In comparison to the other
methods with a local representation of the fields, the Fourier pseudospectral
method has a global representation of the fields using sines and cosines. As a
result, derivatives of fields can be calculated using fast Fourier transforms
(FFTs); in the reciprocal space, a spatial derivative becomes a multiplication
with a wavenumber. This FFT-centric approach avoids the need for linear solvers
that the other spatial discretization may need, leading to a popular
semi-implicit time integration scheme (see below for more details). Fourier
pseudospectral methods have the fastest spatial convergence of any scheme here,
exponential convergence (as compared to polynomial convergence for the other
methods). To balance these advantages, Fourier pseudospectral methods have
restrictions that the other methods do not. The reliance on FFTs means that
periodic boundary conditions are the only type of boundary conditions
possible. The FFTs also require a uniform, structured mesh, so no adaptive
meshing is possible in most situations (e.g., unless a coordinate tranformation
is possible to project onto a uniform, structured mesh).

#### Spatial Discretization Summary Table

| Method | Strengths | Weaknesses |
| ---                   | :---: | :---: |
| __Finite difference__ | Simple to code <br/> Minimal computational overhead <br/> Adaptive meshing is possible | Conservation is not guaranteed depending on the stencil <br/> Difficult to use on complex geometries (more complicated on non-uniform grids and much more complicated (impossible?) on unstructured grids) <br/> Adaptive meshing is constrained by need for a structured mesh |
| __Finite volume__ | Naturally conservative <br/> Adaptive meshing is possible <br/> Can be used for arbitrary geometries (via unstructured meshes) <br/> Simpler to implement than finite element methods  | More complicated to implement than finite difference or Fourier-pseudospectral <br/> Orders of accuracy higher than two substantially increase complexity |
| __Finite element__ | Adaptive meshing is possible <br/> Can be used for arbitrary geometries (via unstructured meshes) <br/> Fairly straightforward extension to higher orders of accuracy <br/> Popularity in other domains may aid multiphysics coupling | Most complicated to code <br/> Most computational overhead |
| __Fourier pseudospectral__ | Simple to code <br/> Highest spatial error convergence rate <br/> "Free" implicit handling of linear terms | Limited to periodic boundary conditions <br/> Limited to uniform, structured meshes |

#### Adaptive Mesh Refinement

Adaptive mesh refinement (AMR) is the process of changing the mesh for a
calculation so that the mesh is finer in regions where the spatial error is
expected to be high (e.g., where fields are changing rapidly) and coarser where
the spatial error is expected to be low (e.g., where fields are changing
slowly). An example of adaptive mesh refinement for the PFHub Benchmark 3
problem is shown below:

(Insert BM3 AMR example)

Note that in this example, the mesh is the finest near the solid-liquid
interface and coarsest far from the interface in the liquid. PFHub Benchmark 3
is an example of a problem where adaptive meshing is particularly useful -- at
any given time, the area of the interfacial region is a small fraction of the
total area. The interfacial region needs a fine mesh to resolve the transition
of the order parameter, but a coarse mesh can be used far into the liquid where
only thermal diffusion is active. The [uploaded
results](https://pages.nist.gov/pfhub/simulations/3a.1/) with by far the
fastest run times for this simulations, using MOOSE and PRISMS-PF, both take
advantage of adaptive meshing.

In other cases, the benefits of adaptive mesh refinement may not be worth the
computation overhead and increase in complexity. The early stages of evolution
for PFHub Benchmark 1 are an example of this case. The composition field is
rapidly varying almost everywhere in the domain, and coarsening the mesh in any
location would likely introduce substantial error.

Three main adaptive meshing schemes are:

- Adaptive unstructured meshes (full freedom of mesh shape, but high overhead)
- Block-structured meshes (a mesh made of rectangular regions at different
  levels of refinement)
- Quadtree/octree meshes (hierarchical mesh where refinement occurs by
  bisecting parent elements)

#### Methods for Applying Boundary Conditions on Internal Boundaries

Although many phase-field problems of interest can be solved in a rectangular
domain with boundary conditions applied on the boundaries of the computational
domain, in some cases, you may want to apply boundary conditions along an
_internal_ boundary. There are two typical circumstances where this occurs. One
circumstance is where the domain of interest has some complex (perhaps dynamic)
shape but you do not want to have a body-fitting mesh (e.g., due to code or
method limitations). The second circumstance is that you are using a Fourier
pseudospectral method and want to impose non-periodic boundary conditions.

Two methods have been developed to handle this issue -- to impose boundary
conditions internally in a computational domain that decouple the solution
domain from the computational domain. The [immersed interface
method](https://epubs.siam.org/doi/book/10.1137/1.9780898717464) is a
sharp-interface method for imposing internal boundary conditions. The [smoothed
boundary method](https://doi.org/10.1088/0965-0393/20/7/075008) is a diffuse
interface method for imposing internal boundary conditions. In some
circumstances, these methods may provide a performant way to side-step the
restrictions on the meshes for Fourier pseudospectral or finite difference
phase-field codes.

To date, none of the PFHub uploads have used these methods, but Benchmark 6b is
an example of where it may be appropriate.

### Choosing a Time Integration Method

If your model is time dependent, you will need to choose a scheme for
integrating in time (or time stepping). A time integrator is typically
described in terms of its order of accuracy and whether it is explicit or
implicit. Classic examples include:

- Forward Euler (FE) method, a first-order explicit method
- Backward Euler (BE) method, a first-order implicit method
- Crank-Nicolson (CN) method, a second-order semi-implicit method

These methods often serve as prototypes for other families of general-purpose
time integrators that have successfully been employed in phase-field codes.
**Many of the community codes that participate in PFHub already offer
selections of time integrators; the documentation of those packages will often
include suggestions on what to choose for your model.**

If you are using a code that does not provide its own time integrators or you
are writing your own code, there are many factors to consider when picking a
scheme.  **The "best" time integrator for your model will depend upon your
governing equations, the spatial discretization, any required linear and
nonlinear solvers, and even the computational hardware you have available.  You
may need to experiment with different schemes to find the right balance of
performance and accuracy for your needs.**  This last step can require
significant amounts of time and coding.

If you need to implement your own time integrator, we provide a brief overview
of integration schemes and their strengths and weaknesses, but this discussion
serves as a relatively high-level overview.  Time integration is an active area
of research across the phase-field, applied math, and computational science
communities, and it is impossible to cover every consideration.

#### Explicit vs. Implicit Methods

In explicit methods, the solution at a future time step is only a function of
previous time steps. By comparison, for implicit methods, the solution at a
future time step is a function of the future and previous time steps. Forward
Euler, for example, has the form $(u^{n+1} - u^n) / \Delta t = f(u^n)$, and
Backward Euler has the form $(u^{n+1} - u^n) / \Delta t = f(u^{n+1})$.
Broadly, we can summarize the strengths (+) and weaknesses (-) as follows:

|                 | Explicit | Implicit |
| ---             | :---:    | :---:    |
| Code complexity | +        | -        |
| Time step cost  | +        | -        |
| Time step size  | -        | +        |
| Memory usage    | +        | -        |

However, there are notable exceptions for each row.  Additionally, choices that
influence each row of the above table also affect the overall consistency of a
scheme.

##### Complexity

In general, explicit solvers do not require nonlinear solvers for the
time-dependent portions of the model, and linear solvers are either not
required (finite difference methods, many finite volume methods) or only
require the inversion of the mass matrix (finite element methods).  This
reduction in necessary solver components can reduce overall development time
(less code, less debugging), which may allow you to start running simulations
sooner.  Additionally, from a working FE code, it is straightforward to
implement either explicit/embedded Runge-Kutta methods, Adams-Bashforth
methods, or Predictor-Corrector (e.g., Milne) methods.

By comparison, fully implicit methods will usually require the implementation
of nonlinear and linear solvers.  Some methods might allow for one of these
eliminated: for example, the linearly implicit Rosenbrock Methods do not
require a nonlinear solver.  However, most codes will adopt a modified Newton
method where a linear solver is required at each nonlinear iteration.  The BE
method can be a prototype for various flavors of implicit Runge-Kutta methods
(IRK, DIRK, SDIRK, ESDIRK, etc.), Adams-Moulton methods, or Backward
Differentiation Formula (BDF) methods, but there are fewer similarities between
these methods than there is between the explicit methods.

##### Time step cost and size

Due to the need to solve a (non-)linear system of equations, one time step of
an implicit method can be orders of magnitude more expensive than one time step
of an explicit method.  If iterative solvers are employed, the cost of each
step will also vary with the overall conditioning of the system from one time
step to the next.

However, it is well-known that explicit solvers often have strict requirements
for the stable time step size (e.g., $\Delta t \propto \Delta x^2$ for FE
discretization of heat equation).  By comparison, most implicit methods are
either unconditionally stable or have much wider stability regions For a given
time interval and similar order of accuracy, an explicit method will usually
take many more time steps than an implicit method, which can increase the
accumulated roundoff error (this will depend on the particular schemes).  Also,
if the increase in stable step size is large enough, the implicit method may
end up requiring less walltime to solve the entire model.

##### Memory usage

Explicit and implicit schemes can have different requirements on the amount of
RAM that is needed for the same overall problem size. If we consider the FE
and BE methods, both can be implemented with two vectors: one for $u^n$ and one
for $u^{n+1}$.  For the explicit method, this places no restrictions on the
resulting solver algorithm: you can solve for $u^{n+1}$ and then either copy it
to $u^n$ or swap the associated pointers in the code.  However, economizing
memory in the implicit scheme limits your method to classical iterative methods
like Newton-Gauss-Seidel, which may not offer satisfactory performance in your
overall code.  More powerful solvers like Newton-Krylov methods will require
more memory, but will allow the implicit method to converge quickly.

While not universally true, this leads to a rule of thumb that for equal
numbers of stages or steps, an explicit scheme will require less memory than an
implicit scheme. However, low-order implicit schemes may be able to take
significantly larger time steps than even high-order explicit schemes, which
can equalize the overall cost.

#### Coupling and Consistency

Comment about how hard it can be to solve multiple equations simultaneously.
Do you solve multiple small systems or one large system?

#### Semi-Implicit and ImEx Methods

In semi-implicit and implicit-explicit (ImEx) methods, different terms in the
governing equations are approximated by different implicit or explicit schemes.
These methods have a long history in the phase-field community and, as
suggested by their name, behave "in between" explicit and implicit schemes.
For example, convex splitting methods for the Cahn-Hilliard equation naturally
result in implicit and explicit terms.  Semi-implicit methods are also the
default choice in Fourier pseudospectral methods.  In more general models,
physics with "fast" time scales may be solved implicitly, while physics with
"slow" time scales may be solved explicitly.

The resulting schemes are usually much cheaper than fully implicit methods.
The largest stable or accurate time step size will typically be reduced
compared to an implicit scheme, but is usually much larger than an explicit
scheme.

#### Time Step Adaptivity

When choosing the time step size of an integrator, there is a need to balance
computational cost (i.e., larger step size, fewer time steps, lower cost) and
the numerical accuracy (smaller step size, better approximation of time
derivatives, lower error, and improved solvability).  Just as we can adaptively
refine and coarsen the mesh of a spatial discretization to improve accuracy or
reduce the computational cost, we can also employ adaptive methods when
choosing a time step.  In many Runge-Kutta and Rosenbrock methods, the scheme
provides an embedded estimate of the error in the solution.  A weighted norm of
this error can be compared to a specified tolerance, which decides if the time
step is accepted or rejected, and then the step size can be adjusted by a PID
controller or some other filter.  In multistep methods, we can derive estimates
of the leading truncation error, which then allows us to check the accuracy of
a step.  However, different methods will have different rules about how to
change step size: Runge-Kutta methods can usually change the step size by large
amounts every time step, but multistep methods often have to change by smaller
amounts or only every few time steps.

When properly implemented, adaptive time stepping improves confidence in
solution quality, minimizes the chance of your solution "exploding" from too
large of a time step, and can save time with expensive schemes by reducing step
size in fast-changing regions of time and increasing it when the dynamics are
slower.

#### General Guidelines

1. For initial development, it is best to start with simpler schemes like
   Forward Euler. This will allow you to gain an understanding of your model's
   behavior, check for bugs in the basic mechanics of the code, and start
   estimating the cost of simulations. If the model is inexpensive or you don't
   plan to use the code more than a few times, a basic integrator may be all
   you need.
1. If you are forced to use small time steps on the scale of time you want to
   simulate, you might consider an implicit integrator (again, start with
   simpler schemes like BE or CN).
1. If your model contains varying time scales (e.g. from coupled physics,
   concentration-dependent mobility) and you have difficulty maintaining a
   stable time step or find that the convergence is unpredictable, adaptive
   time stepping will likely be helpful. If you are using explicit schemes,
   consider trying an embedded Runge-Kutta method. For implicit schemes, you
   might try a higher-order BDF method if your right-hand-side function is
   expensive to compute or a diagonally implicit Runge-Kutta method.
1. If you need to solve an elliptic governing equation simultaneously ([as in
   Benchmark
   6](https://pages.nist.gov/pfhub/benchmarks/benchmark6-hackathon.ipynb/)),
   the resulting system of differential-algebraic equations (DAEs) is likely to
   be very stiff. Even if the time-dependent portion is not expensive to
   evaluate, the cost of constantly performing the elliptic solve can be
   significant with explicit methods. Consider BDF methods, diagonally implicit
   or additive Runge-Kutta methods, or Rosenbrock methods.
1. For coupled physics, carefully examine how each equation is related. Some
   time-dependent terms may be less strongly coupled than others or be more
   costly to evaluate in a fully implicit scheme, so you might consider a
   semi-implicit or ImEx scheme such as an additive Runge-Kutta method.

Many families of time integrators have variants that can choose the time step
size adaptively. Additionally, the Adams and BDF methods have implementations
with variable order of accuracy. **If your code does not already provide time
integration, many packages exist for various flavors of time steppers**,
including but not limited to [SUNDIALS](https://github.com/LLNL/sundials),
[PETSc TS](https://petsc.org/release/docs/manualpages/TS/index.html),
[Tempus](https://trilinos.github.io/tempus.html), and
[DifferentialEquations.jl](https://github.com/SciML/DifferentialEquations.jl).

### Choosing Linear and Nonlinear Solvers (If Needed)

Depending on your spatial/temporal discretizations and whether you need to
solve any elliptic governing equations, you may need to implement or choose a
set of appropriate linear and nonlinear solvers.  **Many codes will already
provide solvers that work well for their packages.**  For example, if you were
implementing a model in FEniCS, the developers have already exposed options
from PETSC's nonlinear and linear solver interfaces and you can easily switch
between different dense, sparse, and iterative methods.  Even if you are
implementing your own code, **solvers may already be available from the
libraries you link to and you may not need to write your own.**  For example,
SUNDIALS provides a set of linear and nonlinear solvers that are already
optimized for its time-stepping routines.

If you need to write your own solvers, we provide some general guidelines
below.

#### When You Might Need Linear or Nonlinear Solvers

The need for (non)linear solvers in a given code will depend on the equations
you are solving and your choices of spatiotemporal discretizations.  As a
simple example, we can consider incremental changes to a 1D Allen-Cahn-type
model for a driven phase transformation (i.e., a simplified form of BM3).  For
a spatial discretization with centered finite differences and time integration
by Forward Euler, the equation at each point of the mesh will be of the form

$$
\phi_i^{n+1} = \phi_i^n + \Delta t ( \phi_{i+1}^n - 2 \phi_i^n + \phi_{i-1}^n - g'(\phi_i^n) - Q p'(\phi_i^n) )
$$

where $n$ corresponds to the time step of the solution, $i$ corresponds to the
grid points, $\Delta t$ is the time step, $Q$ is the driving force, and $g'$
and $p'$ are the derivatives of the double-well and interpolation function,
respectively.  Note that we assume values of 1 for mobility, $\kappa / \Delta
x^2$, and the well height.  All of the quantities on the right-hand side are
known: we can evaluate each $\phi_i^{n+1}$ directly without needing to perform
any linear or nonlinear solvers.  This is sometimes referred to as
"linearization through explicit time stepping."

A common approach to improve time step stability of a PFM is to employ a convex
split of the governing equations where we decompose the nonlinear terms as
$g(\phi) = g_+ (\phi) + g_- (\phi)$ and $p(\phi) = p_+ (\phi) + p_- (\phi)$,
where $+$ indicates the purely convex portion of the function and $-$ is the
remaining (presumably concave) portion.  Typically, we solve this convex-split
approach by ImEx methods such as Forward-Backward Euler:

$$
\phi_i^{n+1} - \Delta t ( \phi_{i+1}^{n+1} - 2\phi_i^{n+1} + \phi_{i-1}^{n+1} - g_+'(\phi_i^{n+1}) - Qp_+'(\phi_i^{n+1}) ) = \phi_i^n - \Delta t ( g_-'(\phi_i^n) + Qp_-'(\phi_i^n) )
$$

This system of equations is now semi-implicit in nature, but we can form the
convex split such that the left-hand side is fully linear, e.g., $g_+(\phi) = a
\phi^2$ for $g(\phi) = \phi^2 (1-\phi)^2$.  As a result, we need a linear
solver for this scheme (such as a tridiagonal LU decomposition) to obtain
$\{\phi^{n+1}\}$, but no nonlinear solver.  We can use larger time step sizes
than with Forward Euler, but we can easily "break" the system so that we don't
observe a monotonic decrease in free energy.

Lastly, we may choose to discretize this system by the Backward Euler method:

$$
\phi_i^{n+1} - \Delta t (\phi_{i+1}^{n+1} - 2\phi_i^{n+1} + \phi_{i-1}^{n+1} - g'(\phi_i^{n+1}) - Qp'(\phi_i^{n+1})  = \phi_i^n
$$

The left-hand side is now fully implicit and nonlinear. We will need to perform
a nonlinear solve at each time step, and this may or may not require the
solution of additional linear systems.

#### Types of Nonlinear Solvers

Two of the most common types of nonlinear methods you will encounter are
**fixed-point methods**, which solve problems of the form $G(u) = u$, and
**root finding methods**, which solve problems of the form $F(u) = 0$.  We can
usually convert fixed-point problems to root-finding problems by letting $F(u)
= G(u) - u$, but the reverse conversion may not be as well-behaved and will be
problem dependent.  These methods have some similarities in underlying
concepts, but the resulting algorithms and implementation can vastly differ.

While not discussed here, another important class of nonlinear methods is Full
Approximation Scheme (FAS) multigrid solvers.  In FAS, we obtain solutions of
the system $A(u) = f$ by relaxing the residual and its error on progressively
coarsened grids, after which we interpolate a set of corrections onto the finer
grid levels.  Depending on the forms of $A(u)$ and $f$, this method can look
like a fixed-point or root-finding method that can be applied to a wide range
of systems, although the implementation can be quite complex.

**Fixed-Point Methods:** A basic fixed-point method will often adopt the
following approach:

1. For the current guess of $u$, compute the nonlinear function, $G(u)$
1. Calculate a new guess by $\tilde{u} = G(u)$
1. Converged if $||u - \tilde{u}|| < tol$
1. Assign $u = \tilde{u}$
1. If not converged, go to (1)

This is often referred to as functional iteration or Picard iteration.  **While
simple to implement, this basic scheme may converge slowly or fail to converge
at all.**  There are schemes to accelerate the convergence of the series (e.g.,
Anderson Mixing, Aitken's Delta-Squared Method), or we can linearize the
nonlinear function about $\tilde{u}$ as $G(\tilde{u}) \approx G(u) + \mathbf{J}
(\tilde{u} - u)$, where $\mathbf{J} = \nabla G(u)$ is the Jacobian matrix of
the nonlinear system.  The resulting **linearized fixed-point method** is then
of the form:

1. For the current guess of $u$, compute the nonlinear function, $G(u)$
1. Solve the linear system $(\mathbf{I} - \mathbf{J})\tilde{u} = G(u) - \mathbf{J}u$
1. Converged if $||u - \tilde{u}|| < tol$
1. Assign $u = \tilde{u}$
1. If not converged, go to (1)

As a note, we may not need to explicitly assemble $\mathbf{J}$, or the
associated linear solver in step (2) might arise "naturally" in the governing
equations.  For example, a pointwise Jacobi-type fixed-point iteration for the
fully-implicit 1D Allen-Cahn example can be written as

$$
\tilde{\phi_i}^{n+1} = \frac{\phi_i^n + \Delta t (\phi_{i+1}^{n+1} + \phi_{i-1}^{n+1} - g'(\phi_i^{n+1}) + g''(\phi_i^{n+1})\phi_i^{n+1} - Q p'(\phi_i^{n+1}) + Q p''(\phi_i^{n+1}) \phi_i^{n+1} )}{1 + 2 \Delta t + \Delta t ( g''(\phi_i^{n+1}) + Q p''(\phi_i^{n+1}) )}
$$

where $g''$ and $p''$ are second derivatives of the respective functions. This
combines steps (1) and (2) into a single process as we sweep over the entire
mesh to evaluate $\{\tilde{\phi}^{n+1}\}$.

**Root-Finding Methods:** In many cases, a root-finding method will often adopt
some sort of Newton-type method where we linearize the function through a
Taylor expansion, obtain a correction to the solution, and then iterate until
convergence.  For example, the classic **Newton-Raphson method** can be written
as:

1. For the current guess of $u$, calculate $F(u)$ and its Jacobian matrix $\mathbf{J}$
1. Correct the solution by solving the linear system $\delta u = - \mathbf{J}^{-1} F(u)$
1. Converged if $||\delta u|| < tol$
1. Assemble the next guess as $\tilde{u} = u + \delta u$
1. Assign $u = \tilde{u}$
1. If not converged, go to (1)

Immediately, we can recognize a similarity between the Newton-Raphson approach
and the linearized fixed-point iteration: both perform a linearization about
the current value of the function for a guess of $u$ and solve a linear system
to obtain the next guess.  (Note: this is part of why we say that fixed-point
solutions can be obtained through an equivalent root-finding method.)

Assembling $\mathbf{J}$ and solving the linear system for $\delta u$ can be
exceptionally expensive. However, if *iterative linear solvers* are employed,
a few iterations of the linear method can significantly reduce the residual of
the nonlinear system.  This leads to the formation of an **inexact Newton
method**:

1. For the current guess of $u$, calculate $F(u)$ and its Jacobian matrix
   $\mathbf{J}$
1. Perform $k$ iterations of an iterative linear solver to approximately solve
   $\mathbf{J} \delta u = - F(u)$
1. Converged if $||\delta u|| < tol$
1. Assemble the next guess as $\tilde{u} = u + \delta u$
1. Assign $u = \tilde{u}$
1. If not converged, go to (1)

We may make further modifications to this algorithm to potentially improve
performance, such as only evaluating $\mathbf{J}$ every few iterations. If we
employ Krylov methods for step (2), we do not necessarily need to form the
Jacobian and instead only need to evaluate matrix-vector products $\mathbf{J}
\delta u$. This leads to the popular and powerful **Jacobian-Free Newton-Krylov
(JFNK) method**, which is employed by many community codes and software
libraries.  Also common are **Newton-Multigrid methods**, where a few multigrid
iterations are performed in step (2).  We may further try to "globalize" the
solution by combining the inexact Newton method with a line-search method to
find an optimal decrease in the residual, usually under the condition $||F(u +
\alpha \delta u)|| < ||F(u)||$ with $\alpha$ between 0 and 1.

If we examine the pointwise Jacobi scheme for the fully-implicit Allen-Cahn, we
see that this is essentially an inexact Newton method that does one inner
iteration of Jacobi relaxation at each outer Newton iteration.  Thus, we can
see that there are some general similarities between these broad classes of
nonlinear methods.  **However, it is rare that the resulting algorithms are
exactly equivalent to each other, and it can be nontrivial to switch from a
fixed-point to a root-finding solver.**

#### Types of Linear Solvers

In the above discussion, we've observed that linear solvers are necessary as a
result of the discretization of the governing equations or because we need to
solve a nonlinear system of equations. **The choice of solver will depend on
the size of your simulation and whether you can easily evaluate a Jacobian
matrix of your system.**  Up to a few thousand unknowns, direct linear solvers
(e.g. LAPACK, UMFPACK, cuSolver, SuperLU) that offer dense or sparse variants
of typical methods like LU decomposition and QR factorization will often be
efficient, provided it is easy to evaluate the entries of the Jacobian matrix.
However, it is important to remember that dense direct solvers typically scale
O(n^2) for required memory and O(n^3) for the number of operations and quickly
become unsolvable.  The scalability of sparse direct solvers will depend on the
particular method and the implementing library.  For millions or billions of
unknowns across large numbers of CPUs or GPUs, iterative solvers are often
necessary.  Here, a wide range of options exists; in particular, relaxation,
multigrid, and Krylov methods all parallelize well and have been employed in a
variety of community codes and numerical libraries.

Some of the simplest iterative methods are stationary relaxation methods such
as the Jacobi, Gauss-Seidel, and (Symmetric) Successive Over-Relaxation
methods.  Classically, these are derived by splitting the Jacobian matrix as $J
= D - E - F$, where $D$ is the diagonal, $-E$ is the purely lower triangular
portion, and $-F$ is the purely upper triangular portion.  Depending on how we
manipulate $D$, $E$, and $F$, we obtain the above methods or their symmetric
variants; however, **we may not need to directly form the Jacobian or its
submatrices**, as we observed in the above Allen-Cahn example.  **The
simplicity and low memory requirements of relaxation methods make them
attractive methods, but they often converge slowly.**  The convergence can be
improved by employing multigrid methods: **Geometric Multigrid methods converge
rapidly by applying relaxation methods** on multiple levels of structured grids
and are popular solvers for a wide range of problems.

Krylov methods (e.g. Conjugate Gradient, GMRES, BiCGSTAB, TFQMR) are a powerful
family of linear solvers that form a subspace from a linear operator (such as
the Jacobian) and the residual vectors of the system (this is a very simplified
explanation). While these methods can be formulated to directly use the
Jacobian, **most only require the evaluation of Jacobian-vector products**,
which is what leads to the "Jacobian-free" nature of JFNK methods.  This
Jacobian-vector product is typically evaluated in either a functional form if
the Jacobian can be easily obtained (e.g., the kernel of a linear heat
equation), but for methods such as JFNK, **the Jacobian-vector product is
readily approximated by the Gateaux derivative**.  These methods can,
therefore, be **straightforward to implement, even with intricate linear
functions**.  However, **convergence is strongly dependent on the condition
number of the operator matrix.**

The performance of Krylov methods can be improved through **preconditioning**.
Here, we form an approximation to the overall Jacobian matrix, $\mathbf{M}
\approx \mathbf{J}$.  Depending on the exact method, we will then find
solutions to some linear system of the residual, e.g., $\mathbf{M} z = r$, at
certain points of the algorithm.  If done correctly, the computational time and
the number of iterations to reach the desired tolerance will decrease.  The
preconditioner, therefore, needs to (1) be easy to form and assemble, (2) be
easy to solve and apply, and (3) should capture the essential physics of the
system.  **Designing preconditioners is problem-dependent and can have a huge
impact on the overall model performance**.  Many possible options have been
applied by the phase-field community, including relaxation methods (e.g.,
Jacobi, Gauss-Seidel), multigrid methods (both geometric and algebraic),
incomplete LU and Cholesky factorizations, sparse approximate inverses, and
multiplicative or additive Schwarz methods.  **You will likely need to
experiment with the best preconditioner for your model,** which can require a
significant amount of time if your libraries don't already provide them.

In summary: when choosing linear solvers, **it is not trivial to switch from a
relaxation or multigrid method to a Krylov method (or vice versa)**, although
the former methods can be used as preconditioners for the latter.  Changing
from one Krylov method to another can be relatively straightforward, but **some
Krylov methods only apply to symmetric positive definite matrices**.  Modifying
the preconditioner can have varying difficulties: switching from a Jacobi to a
Gauss-Seidel preconditioner is trivial, but switching to an ILU preconditioner
might require much more effort.

### Choosing an appropriate parallelism approach

(Discuss distributed vs shared, CPU vs GPU)

#### Distributed memory

(pretty much just MPI, also mention UPC or Legion or Charm++?)

#### Shared memory

CPU/GPU, performance portability layers

## Choosing appropriate numerical libraries and/or what to write yourself

The phase-field and broader scientific computing community have developed many
libraries that you may choose to use as you implement your phase-field
model. As you decide which methods you want to use, an important consideration
is whether you intend to write code for certain functionality yourself or
whether you want to use an existing library/framework for that functionality.

### Investigate existing solutions before writing your own

Before coding your own functionality, it is always a good idea to explore what
solutions already exist. Researchers have devoted their careers to developing
some scientific computing libraries and frameworks, and they may be better for
your application than something you write yourself. Even in the cases where you
_could_ write code for some functionality yourself that is an improvement over
existing solutions, it may not be worth your time to actually do so. Finally,
even if you decide that your best course is to write your own functionality,
learning about other solutions may inform your approach and/or help you frame
to potential users what distinguishes your implementation.

#### Existing phase-field codes and frameworks

PFHub maintains a [list of open source phase-field codes and
frameworks](https://pages.nist.gov/pfhub/codes/), one or more of which may fit
your needs.

#### Consider how existing libraries interact

Some libraries are packages in a larger collection where interfaces for
interoperability have already been developed. Such large collections include
[Trilinos](https://trilinos.github.io/), [PETSc](https://petsc.org/release/),
and
[hypre](https://github.com/hypre-space/hypre)/[SAMRAI](https://github.com/LLNL/SAMRAI)/[SUNDIALS](https://sundials.readthedocs.io/en/latest/). Staying
within an ecosystem may reduce your time to solution and will keep you on a
well-traveled path if you reach out to developers or a user community for help.

#### Consider local expertise

An important consideration for choosing a library or framework is who else uses
it. If many other researchers in your organization use a particular
library/framework, you may want to as well -- when you run into issues, they
might be able to help you (and vice versa).

### Consider licensing

Another consideration for choosing a library or framework is what the license
terms are. For commercial software, license terms may dictate that a new
license must be bought for each CPU core being used. With paid software, you
may also want to consider the future of your work, even if you have access to
that software now, that may not be true if you move institutions, your
institution changes what they buy, etc. Other people who would be interested in
using code that you develop would also have to pay for the underlying software.

For open-source libraries and frameworks, a wide variety of open-source
licenses exist. While a discussion of the differences between these licenses is
out of the scope of this document, the details of the license may constrain how
you can use the library/framework and how you can distribute derivative
works. One reference for open-source licenses is the [Open Source
Initiative](https://opensource.org/licenses). The terms of these licenses can
be confusing and disputed. The legal department at your institution may be able
to offer advice.


## Further reading

### Solver Methods:

1. Y. Saad. *Iterative Methods for Sparse Linear Systems, Second Edition.*
   Society for Industrial and Applied Mechanics, 2003.
1. W.L. Briggs, V. Emden Henson, S.F. McCormick. *A Multigrid Tutorial, Second
   Edition.* Society for Industrial and Applied Mechanics, 2000.
1. D.A. Knoll, D.E. Keyes. "Jacobian-free Newton-Krylov methods: a survey of
   approaches and applications." *J. Comput. Phys.* 193 (2004) 357-397.

***

Topics we want to move to other sections:

- Conservation laws and symmetries (Model Formulation)
- Have an expectation going in (Model Formulation)
- Add pieces of the model one part at a time during debugging (Software Development)
- Ensure physical expectations are respected (Software Development)

***

<!--
OLD TEXT

Contributors: Stephen DeWitt, Alex Chadwick

Alex Chadwick, Steve Dewitt

* Carefully consider your spatial discretization scheme (FFT, FD, FV, FEM)
* Consider your temporal discretization scheme and use temporal error
  estimation when possible (implicit, explicit, etc.)
* Identify relevant conservation laws and symmetries for your model and
  carefully ensure they are respected even after long times
* Start small and add complexity. Add pieces to the model one part at a time
  during debugging. Check to see if it is working, then add more, then test
  again.
* Have an expectation going in (back of the envelope, consider basic physics),
  and if it doesn’t do that, figure out why.
-->

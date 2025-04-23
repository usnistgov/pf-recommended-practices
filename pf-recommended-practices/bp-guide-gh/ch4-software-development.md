(label-software-development)=
# Software Development

Daniel Schwen ([@dschwen](https://github.com/dschwen)), Jon Guyer ([@guyer](https://github.com/guyer)), Trevor Keller ([@tkphd](https://github.com/tkphd))

People don't usually arrive at phase-field methods from a software development
background.  This page recommends some approaches to reduce errors and wasted
time before reaching a usable state -- including the installation process.

* Start small and ensure quality before getting bigger
* Consider whether to invoke an existing library or implementation to save
  time, or write it yourself to gain a deeper understanding or implement
  non-standard features

## Follow good coding practices

The choice of the programming language requires a multitude of considerations

* Ease of development, familiarity of the language by the development team
* Target architecture and operating systems, portability
* Use of parallel computing infrastructures, MPI, GPU support
* Performance of the resulting code, compiled, interpreted, hybrid approaches

### Commenting code and design

Comment code generously. Whether it is for other developers working on the code
or for yourself a couple of years (months?) in the future.

> "Code is more often read than written."

Documentation can be roughly split into API documentation and inline code documentation.

* API documentation facilitates reuse of the code.
  - [Doxygen](https://doxygen.nl/) is the _de facto_ standard for C++ API
    documentation, but supports many other languages, such as C, C#, Java,
    and Python.
  - [Sphinx](https://www.sphinx-doc.org/) is the _de facto_ standard for Python
    API documentation.
  - [ReadTheDocs](https://readthedocs.org/) is a widely used hosting site for
    software documentation.
* Inline code comments improve the readability and should capture the intent of
  the documented code (which can be very valuable for debugging purposes).
  - Choose variable names that reflect their function, and don't be afraid of
    long names.
  - Use longer comments and [docstrings](https://peps.python.org/pep-0257/) to
    provide context for larger blocks of code.

API documentation should be augmented by _design documents_ that offer a high
level overview over a code, library, or framework. Design documentation
describes the way different pars of the code fit together and their roles in
the overall project.

### End-user documentation

Key to successful software projects with broad adoption is good end-user
documentation. To ease the learning curve for new users, this documentation
should contain

* Installation
  - Hardware requirements and software dependencies
  - How to build
  - How to install
  - HPC considerations
  - "Run the docker image" is not a (complete) installation instruction
* Introductory tutorials
* Examples
* Theory docs
* Reference documentation

(label-version-control-and-metadata)=
### Version control and metadata

To facilitate contributions to a code, version control is
indispensable. Version control keeps track of all changes to the code and keeps
every team member working on the right version. Version control systems enable
the use of code review workflows (such as the _pull request_ workflow on
[GitHub](https://github.com). Code review in turn allows for input from
multiple developers and for productive feedback and improvement cycles leading
to a higher code quality. Open version control platforms also invite outside
contributions and user engagement, leading to project growth.

On open platforms, it is important to add and maintain metadata, such as the
code license. Authorship is tracked through the version control system.

(label-software-license-considerations)=
#### License considerations

> [Largely not unique to phase field, or even to scientific computing, so not a lot to say here, but should reference good guidance]

* If you do not spell out your license, potential users must assume you are
  granting _no rights_ to copy or reuse it.
* If your institution stipulates a license, use it.
* If choosing your own, understand the implications of the license you apply:
  - Some licenses may be incompatible with institutional requirements
  - Some licenses, e.g., GPL, may be an obstacle to commercial adoption
* Don't try to write your own!
* http://choosealicense.org
* https://tldrlegal.com

### Continuous integration and testing

Strive for a large test coverage of your code. Ideally, tests should be added
to your project that exercise every single line of written code. The utility of
a thorough test coverage is twofold:

1. It protects against regressions, i.e., unintended changes in code
   behavior. This in turn liberates developers to refactor code as needed to
   improve on code design and code quality.
2. Tests can be used by the end-users to verify that they have built a fully
   working version of the code.

Tests can come in the form of small problems (e.g. mini phase field problems)
or unit tests. Unit tests do not build a complete simulation, but allow
targeted and fast testing of select components of a code. In practice, a mix of
execution of the full code and unit tests will be required to achieve
comprehensive coverage.

Use a continuous integration (CI) infrastructure. CI systems can connect to
version control systems and trigger automatic builds and execution of test
suites in your development workflow. For example, testing should occur for
every pull request that is made to a code, to ensure that the software is never
put into a broken state by incorporating new changes.

The optimal CI infrastructure builds and tests the project with all explicitly
supported compilers on all explicitly supported operating systems. In practice,
the resulting matrix can be very large in particular when it comes to multiple
versions of supported compilers/toolchains, which can be mitigated by declaring
and testing a minimal version for each compiler in addition to a current
version.

### Code verification

While regression tests only guarantee to maintain the _status quo_, they do not
guarantee the correctness of the simulation results. _Code verification_ is
required to ensure the code does what it is designed to do. Several
verification approaches exist.

1. Comparison to analytic solutions. This is often only achievable for the
   simplest problems (after all most simulation software is developed precisely
   to _numerically_ solve problems that have no analytical solutions).
2. [Method of manufactured
   solutions](https://www.osti.gov/servlets/purl/759450-wLI4Ux/native/) (also
   known as _forced solutions_). Here, a user defined solution to an equation
   system is forced by applying boundary conditions and source terms derived
   from inserting the user defined solution into the equation system.
3. Scaling laws. Known scaling laws, such as coarsening rate over time or
   convergence rate as a function of domain discretization, can be tested to
   discover implementation errors.

## TODO

- Am I disseminating this software?

# Data Generation and Curation

Authors:

- [Trevor Keller](https://www.nist.gov/people/trevor-keller), NIST, [@tkphd]
- [Daniel Wheeler](https://www.nist.gov/people/daniel-wheeler), NIST, [@wd15]
- [Damien Pinto](https://ca.linkedin.com/in/damien-pinto-4748387b), McGill, [@DamienPinto]
- [Jonathan Guyer](https://www.nist.gov/people/jonathan-e-guyer), NIST, [@guyer]

## Overview

Phase-field models are characterized by a form of PDE related to an Eulerian
free boundary problem and defined by a diffuse interface. Phase-field models
for practical applications require sufficient high fidelity to resolve both the
macro length scale related to the application and the micro length scales
associated with the free boundary. Performing a useful phase-field simulation
requires extensive computational resources and can generate large volumes of
raw field data. This data consists of field variables defined throughout a
discretized domain or an interpolation function with many Gauss
points. Typically, data is stored at sufficient temporal frequency to
reconstruct the evolution of the field variables.

In recent years there have been efforts to embed phase-field models into
integrated computational materials engineering (ICME) based materials design
workflows {cite}`Tourret2022`. However, to leverage relevant phase-field
resources for these workflows a systematic approach is required for archiving
and accessing data. Furthermore, it is often difficult for downstream
researchers to find and access raw or minimally processed data from phase-field
studies, before the post-processing steps and final publication. In this
document, we will provide motivation, guidance and a template for packaging and
publishing findable, accessible, interoperable, and reusable (FAIR) data from
phase-field studies as well as managing unpublished raw data
{cite}`Wilkinson2016`. Following the protocols outlined in this guide will
provide downstream researchers with an enhanced capability to use phase-field
as part of larger ICME workflows and, in particular, data intensive usages such
as AI surrogate models. This guide serves as a primer rather than a detailed
reference on scientific data, aiming to stimulate thought and ensure that
phase-field practitioners are aware of the key considerations before initiating
a phase-field study.

## Definitions

It is beneficial for the reader to be clear regarding the main concepts of FAIR
data management when applied to phase-field studies. Broadly speaking, **FAIR
data management** encompasses the curation of simulation workflows (including
the software, data inputs and data outputs) for subsequent researchers or even
machine agents. **FAIR data** concepts for simulation workflows have been well
explained elsewhere, see {cite}`Wilkinson2024`. A **scientific workflow** is
generally conceptualized as a graph of connected actions with various inputs
and outputs. Some of the nodes in a workflow may not be entirely automated and
require human agent inputs, which can increase the complexity of workflow
curation. Workflow nodes include the pre and post-processing steps for
phase-field simulation workflows. In this guide, the **raw and post-processed
data** is considered to be distinct from the **metadata**, which describes the
simulation, associated workflow and data files. The **raw data** is generated
by the simulation as it is running and often consists of temporal field
data. The **post-processed data** consists of derived quantities and images
generated using the **raw data**. The **software** or **software application**
used to perform the simulation generally refers to a particular phase-field
code, which is part of the larger **computational environment**. The **code**
might also refer to the **software**, but the distinction is that the **code**
may have been modified by the researcher and might include **input files** to
the **software application**. Although the code can be considered as data in
the larger sense, in this work, the data curation process excludes
consideration of code curation, which involves its own distinct practices. See
the [Software Development](label-software-development) section of the best
practices guide for a more detailed discussion of software and code curation.

```{mermaid}
---
title: A Phase-Field Workflow
config:
  layout: elk
  elk:
    mergeEdges: true
    nodePlacementStrategy: NETWORK_SIMPLEX
---
flowchart TD

INPUT@{ shape: doc,      label: "Input Files", fill: #f96 }
PARAM@{ shape: in-out,   label: "Parameters" }
PREPR@{ shape: rect,     label: "Pre-processing (e.g. CALPHAD or Meshing)" }
PREDT@{ shape: bow-rect, label: "Pre-processed Data" }

ENVIR@{ shape: in-out,   label: "Computational Environment" }
SOURC@{ shape: in-out,   label: "Code" }
METAD@{ shape: bow-rect, label: "Metadata" }

PFSIM@{ shape: procs,    label: "Phase-Field Simulation" }
FDATA@{ shape: bow-rect, label: "Raw Field Data" }
SCRAT@{ shape: bow-rect, label: "Scratch Data" }

POSPR@{ shape: display,  label: "Post-processing (e.g. Data Visualization)" }
POSDT@{ shape: bow-rect, label: "Post-processed Data" }

CURAT@{ shape: manual,   label: "Curation" }
CRATE@{ shape: disk,     label: "Data Repository" }

PARAM --> INPUT & METAD
SOURC --> PFSIM & METAD
ENVIR --> PFSIM & METAD & PREPR
INPUT --> PREPR & PFSIM & METAD & POSPR
PREPR --> PREDT --> PFSIM
PFSIM --> FDATA & SCRAT
SCRAT --> PFSIM
FDATA --> POSPR --> POSDT

METAD & FDATA & POSDT --> CURAT --> CRATE
```

## Data Generation

Let's first draw the distinction between **data generation** and **data
curation**. Data generation involves writing raw data to disk during the
simulation execution and generating post-processed data from that raw
data. Data curation involves packaging the generated data objects from a
phase-field workflow or study along with sufficient provenance metadata into a
FAIR research object for consumption by subsequent scientific studies.

When performing a phase-field simulation, one must be cognizant of several
factors pertaining to data generation. Generally speaking, the considerations
can be defined as follow.

- Writing raw data to disk
- File formats
- Recovering from crashes and restarts
- Using workflow tools
- High performance computing (HPC) environments and parallel writes

These considerations will be outlined below.

### Writing raw data to disk

Selecting the appropriate data to write to disk during the simulation largely
depends on the requirements such as post-processing or debugging. However, it
is good practice to consider future uses of the data that might not be of
immediate benefit to the research study. Lack of forethought in retaining data
could hinder the data curation of the final research object. The data
generation process should be considered independently from restarts, which is
discussed in a [subsequent section](#label-restarts). In general, the data
required to reconstruct derived quantities or the evolution of field data will
not be the same as the data required to restart a simulation.

On many HPC systems writing to disk frequently can be expensive and
intermittently stall a simulation due to a number off factors such as I/O
contention, see the [HPC section below](#label-hpc-environments).  Generally,
when writing data it is best to use single large writes to disk as opposed to
multiple small writes especially on shared file systems (i.e. "perform more
write bytes per write function call" {cite}`Paul2020`). In practice this could
involve caching multiple field variables across multiple save steps and then
writing to disk as a single data blob in an HDF5 file for example. Caching and
chunking data writes is a trade-off between IO efficiency, data loss due to
jobs crashing, simulation performance, memory usage and communication overhead
for parallel jobs. Overall, it is essential that the IO part of a code is well
profiled using different write configurations. The replicability of writes
should also be tested by checking the hash of data files while varying parallel
configurations, write frequencies and data chunking strategies. I/O performance
can be a major bottleneck for larger parallel simulations, but there are tools
to help characterize I/O, see {cite}`Ather2024` for a thorough overview.

### File formats

As a general rule it is best to choose file formats that work with the tools
already in use and / or that your colleagues are using. There are other
considerations to be aware of though. Human readable formats such as CSV, JSON
or even YAML are often useful for small medium data sets (such as derived
quantities) as some metadata can be embedded alongside the raw data resulting
in a FAIRer data product than standard binary formats. Some binary file formats
also support metadata and might be more useful for final data curation of a
phase-field study even if not used during the research process. One main
benefit of using binary data (beyond saving disk space) is the ability to
preserve full precision for floating point numbers. See the [Working with
Data][working-with-data] section of the Python for Scientific Computing
document for a comparison of binary versus text based formats. The longevity of
file formats should be considered as well. A particularly egregious case of
ignoring longevity would be using the Pickle file format in Python, which is
both language dependent and code dependent. It is an example of data
serialization, which is used mainly for in-process data storage for
asynchronous tasks and checkpointing, but not good for long term data storage.

There are many binary formats used for storing field data based on an Eulerian
mesh or grid. Common formats for field data are NetCDF, VTK, XDMF and
EXODUS. Within the phase-field community, VTK seems to be the mostly widely
used. VTK is actually a visualization library, but supports a number of
different native file formats based on both XML and HDF5 (both non-binary and
binary). The VTK library works well with FE simulations supporting many
different element types as well as parallel data storage for domain
decomposition.  See the [XML file formats documentation][vtk-xml] for VTK for
an overview of the many different file extensions and their meanings. In
contrast to VTK, NetCDF is more geared towards gridded data having arisen from
atmospheric research (using finite difference grids rather than finite element
meshes). For a comparison of performance and metrics for different file types
see the [MeshIO README.md][meshio].

The MeshIO tool {cite}`Schlomer` is a good place to start for IO when writing
custom phase-field codes in Python (or Julia using `pyimport`). MeshIO is also
a good place to start for exploring, debugging or picking apart file data in an
interactive Python environment. Debugging data can be much more difficult with
GUI style data viewers such as Paraview. The scientific Python ecosystem is
very rich with tools for data manipulation and storage such as Pandas, which
supports table data storage in many different formats, and xarray
{cite}`Hoyer2017` for higher dimensional data storage. [xarray supports NetCDF
file storage][xarray-io], which includes coordinate systems and metadata in
HDF5. Both Pandas and xarray can be used in a parallel or a distributed manner
in conjunction with Dask. Dask along with xarray supports writing to the Zarr
data format which supports out-of-memory operations.

(label-restarts)=
### Recovering from crashes and restarts

A study from 2020 of HPC systems calculated the success rate (I.e. no error code
on completion) of multi-node jobs with non-shared memory at between 60% and 70%
{cite}`Kumar2020`. Needless to say that check-pointing is absolutely required
for any jobs of more than a day. Nearly everyday, an HPC platform will
experience some sort of failure {cite}`Benoit2022b`, {cite}`Aupy2014`. That
doesn't mean that every job will fail every day, but it would be optimistic to
think that jobs will go beyond a week without some issues. Given the failure
rate one can estimate how long it might take to run a job without
check-pointing. A very rough estimate for expected completion time assuming
instantaneous restarts and no queuing time is given by,

$$ E(T) = \frac{1}{2} \left(1 + e^{T / \mu} \right) T $$

where $T$ is the nominal job completion time with no failures and $\mu$ is the
mean time to failure. The formula predicts an expected time of 3.8 days for a
job that nominally runs for 3 days with a $\mu$ of one week. The formula is of
course a gross simplification and includes many assumptions that are invalid in
practice (such as a uniform failure distribution), but regardless of the
assumptions the exponential time increase without check-pointing is
inescapable. Assuming that we're agreed on the need for checkpoints, the next
step is to decide on the optimal time interval between checkpoints. This is
given by the well known Young/Daly formula,

$$ W = \sqrt{2 \mu C} $$

where $C$ is the wall time required to execute the code associated with a
checkpoint {cite}`Benoit2022a`, {cite}`BautistaGomez2024`. The Young/Daly
formula accounts for the trade off between the start up time cost for a job to
get back to its original point of failure and the cost associated with writing
the checkpoint to disk. For example, with a weekly failure rate and $C=6$
minutes the optimal write frequency is 5.8 hours. In practice these estimates
for $\mu$ and $C$ might be a little pessimistic, but be aware of the trade off
{cite}`Benoit2022b`. Note that some HPC systems have upper bounds on run
times. The Texas Advanced Computing Center has an upper bound of 7 days for most
jobs so $\mu<7$ days regardless of other system failures.

Given the above theory, what are some practical conclusions to draw?

- Take some time to estimate both $\mu$ and $C$. It might be worth discussing
  the $\mu$ value with the HPC cluster administrator to get some valid
  numbers. Of course $C$ can be estimated by running test jobs. Estimating these
  values can be difficult due to HPC cluster volatility, but it's good to know
  if you should be checkpointing every day or every hour or even never
  checkpointing at all in the circumstances that $W \approx T$.
- Ensure that restarts are deterministic (i.e. results don't change between a
  job that restarts and one that doesn't). One way to do this is to compare
  hashes from raw data output files assuming that the simulation itself is
  deterministic.
- Consider using a checkpointing library if you're using a custom phase-field
  code or even a workflow tool such as Snakemake which has the inbuilt ability
  to handle checkpointing. A tool like Snakemake is good for large parameter
  studies where it is difficult to keep track of a multiplicy of jobs and their
  various output files making restarts complicated. The `pickle` library is
  acceptable for checkpointing Python programs as checkpoint data is only useful
  for a brief period.
- Many PDE solvers and dedicated phase field codes will have a checkpoint
  mechanism built in. However, never trust the veracity of these
  mechanisms. Always run your own tests varying parallel parameters and
  checkpoint frequency!
  
Checkpointing strategies on HPC clusters is a complex topic, see
{cite}`Herault2019` for an overview.

### Using Workflow Tools

In general when running many phase-field jobs for a parameter study or dealing
with many pre and post-processing steps, it is wise to employ a workflow
tool. The authors are particularly familiar with Snakemake so discussion is
slanted towards this tool. One of the main benefits of using a workflow tool is
that the user is more likely to automate workflow steps that ordinarily would
not be automated with ad-hoc tools such as Bash scripts. Workflow tools enforce
a structure on and careful consideration of the inputs, outputs and overall task
graph of the workflow. As a side effect, the imposed graph structure produces a
much FAIRer research object when the research is eventually published. Future
reuse of the study is much easier when the steps in producing the final data
objects are clearly expressed. When using Snakemake, the `Snakefile` itself is a
clear human readable record of the steps required to re-execute the
workflow. Ideally, the `Snakefile` will fully automate all the steps required,
starting from the parameters and raw input data, to reach the final images and
data tables used in any publications. In practice this might be quite difficult
to implement due to the chaotic nature of research projects and the associated
workflows.

A secondary impact of using a workflow tool is that it often imposes a directory
and file structure on the project. For example, Snakemake has an [ideal
suggested directory structure][snakemake-directory]. An example folder structure
when using Snakemake would look like the following.

```
.
├── config
│   └── config.yaml
├── LICENSE.md
├── README.md
├── resources
├── results
│   └── image.png
└── workflow
    ├── envs
    │   ├── env.yaml
    │   ├── flake.lock
    │   ├── flake.nix
    │   ├── poetry.lock
    │   └── pyproject.toml
    ├── notebooks
    │   └── analysis.ipynb
    ├── rules
    │   ├── postprocess.smk
    │   ├── preprocess.smk
    │   └── sim.smk
    ├── scripts
    │   ├── func.py
    │   └── run.py
    └── Snakefile
```

Notice that the above directory structure includes the `envs` directory. This
allows different steps in the workflow to be run with independent computational
environments. Additionally, most workflow tools will support both HPC and local
workstation execution and make porting between systems easier.

See {cite}`Moelder2021` for a more detailed overview of Snakemake and a list of
other good workflow tools.

(label-hpc-environments)=
### HPC Environments and parallel writes

Under construction

## Data Curation

Data curation involves manipulating an assortment of unstructured data files,
scripts and metadata from a research study into a coherent research data object
that satisfies the principles of FAIR data. A robust data curation process is
often a requirement for compliance with funding bodies and to simply meet the
most basic needs of transparency in scientific research. The fundamental steps
to curate a computational research project into a research data object and
publish are as follows.

- **Automation:** Automate the entire computational workflow where possible
  during the research process from initial inputs to final research products
  such as images and data tables.
- **Public Development:** Submit the code and workflows appropriately during
  development. This step will not be described here, but is discussed in the
  [Version control and metadata section](label-version-control-and-metadata) of
  the [Software Development Guide](label-software-development).
- **Metadata Standards:** Employ a suitable metadata standard where possible to
  describe different aspects of the research project such as the raw data files,
  derived data assets, software environments, numerical algorithms and problem
  specification.
- **Licensing:** License the research work appropriately. This may require a
  separate license for the data products as they are generally not archived in
  the code repository.
- **Data Repositories:** Select a data repository to curate the data, submit the
  data and then obtain a DOI.

The above steps are difficult to implement near the conclusion of a research
study. The authors suggest considering these steps at the outset and during a
study and also considering these steps as part of an overall protocol in a
computational materials research group.

### Automation

Automating workflows in computational materials science is useful for many
reasons, however, for data curation purposes it provides and added benefit. In
short, an outlined workflow associated with a curated FAIR object is a primary
method to improve FAIR quality for subsequent researchers. For most workflow
tools, the operation script outlining the workflow graph is the ultimate form of
metadata about how the archived data files are used or generated during the
research. For example, with Snakemake, the `Snakefile` has clearly outlined,
human-readable inputs and outputs as well as the procedure associated with each
input / output pair. The computational environment, command line arguments,
environment variables are recorded for each workflow step as well as the order
of execution of each of these steps.

In recent years there have been efforts in the life sciences to provide a
minimum workflow for independent code execution during the peer review
process. The CODECHECK initiative {cite}`Nuest2021` tries to provide a standard
for executing workflows and a certification if the workflow satisfies basic
criteria. These types of efforts will likely be used within the computational
materials science community in the coming years so adopting automated workflow
tools as part of your research will greatly benefit this process.

- See also {cite}`Leipzig2021`

### Metadata Standards

### Licensing

A license serves two purposes: it articulates ownership of the data
and it describes the conditions for use of the data by others.
Data released without an accompanying license cannot be used by anybody but the author.
Publicly available is not the same thing as "public domain". Data repositories like [Zenodo] require declaration of a license in order to deposit data.

Licensing of data is distinct from [licensing software](label-software-license-considerations).
This is, in part, because, software and data may be released in different places,
e.g., GitHub vs. Zenodo, requiring a license statement in each location.
They may also have different release cycles;
data might be released only at the completion of a research study, and rarely or never updated afterwards,
whereas new versions of code might be publicly posted several times per day when under active development.

Even if released concurrently,
[data and software have different needs][data-software-different-license].
With software, there is a distinction between source code and executable.
A software license must address the conditions for sharing and reuse of
source code. A software license may also address patent rights.

Your institution may stipulate a license; you should use that.

See the [Creative Commons discussion of data licensing considerations][https://wiki.creativecommons.org/wiki/Data].

### Selecting a data repository

Dockstore and Workflowhub https://arxiv.org/pdf/2410.03490

## References

```{bibliography}
:filter: docname in docnames
```

<!-- links -->

[@tkphd]: https://github.com/tkphd
[@wd15]: https://github.com/wd15
[@DamienPinto]: https://github.com/DamienPinto
[@guyer]: https://github.com/guyer
[CodeMeta]: https://codemeta.github.io
[CodeMeta Generator]: https://codemeta.github.io/codemeta-generator/
[FAIR Principles]: https://www.go-fair.org/fair-principles/
[PFHub]: https://pages.nist.gov/pfhub
[PFHub repository on GitHub]: https://github.com/usnistgov/pfhub
[Schema.org]: https://www.schema.org
[Zenodo]: https://zenodo.org
[fair-phase-field]: https://doi.org/10.5281/zenodo.7254581
[schemaorg]: https://github.com/openschemas/schemaorg
[structured data schema]: https://en.wikipedia.org/wiki/Data_model
[link1]: https://workflows.community/groups/fair/best-practices/
[meshio]: https://github.com/nschloe/meshio?tab=readme-ov-file#performance-comparison
[vtk-xml]: https://docs.vtk.org/en/latest/design_documents/VTKFileFormats.html#xml-file-formats
[working-with-data]: https://aaltoscicomp.github.io/python-for-scicomp/work-with-data/#binary-file-formats
[xarray-io]: https://docs.xarray.dev/en/stable/user-guide/io.html
[snakemake-directory]: https://snakemake.readthedocs.io/en/stable/snakefiles/deployment.html
[data-software-different-license]: https://creativecommons.org/faq/#can-i-apply-a-creative-commons-license-to-software

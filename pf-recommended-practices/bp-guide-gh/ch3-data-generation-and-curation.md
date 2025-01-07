# Data Generation and Curation

Authors:

- [Trevor Keller](https://www.nist.gov/people/trevor-keller), NIST, [@tkphd]
- [Daniel Wheeler](https://www.nist.gov/people/daniel-wheeler), NIST, [@wd15]
- [Damien Pinto](https://ca.linkedin.com/in/damien-pinto-4748387b), McGill, [@DamienPinto]

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
---p
flowchart TD
    id1@{   shape: lean-r,   label: "Input Files", fill: #f96 }
    id1.2@{ shape: lean-r,   label: "Parameters" }
    id1.1@{ shape: lean-r,   label: "Code" }
    id2@{   shape: lean-r,   label: "Computational Environment" }
    id2.5@{ shape: rect,     label: "Pre-processing, (e.g. CALPHAD or Meshing)" }
    id2.7@{ shape: bow-rect, label: "Pre-processed Data" }
    id3[Phase-Field Simulation]
    id3.5@{ shape: bow-rect, label: "Scratch Data" }
    id4@{   shape: bow-rect, label: "Raw Field Data" }
    id5@{   shape: rect,     label: "Post-processing (e.g. Data Visualization)" }
    id6@{   shape: bow-rect, label: "Post-processed Data" }
    id7@{   shape: lin-cyl,  label: "Data Repository" }
    id8@{   shape: bow-rect, label: "Metadata" }
    id1.2-->id1
    id1-->id2.5
    id1-->id5
    id2.5-->id2.7-->id3
    id1-->id3
    id1.1-->id3
    id2-->id3
    id3-->id4-->id5-->id6
    id3-->id3.5-->id3
    id2-->id2.5
    id2-->id5
    id6--Curation-->id7
    id4--Curation-->id7
    id8--Curation-->id7
    id1.2-->id8
    id1.1-->id8
    id1-->id8
    id2-->id8
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
to help characterize I/O, see {cite}`Ather2024` for an overview.

### File formats

In general, when running phase-field simulations, the user is limited to the
file format that the software supports. For example, if the research is using
PRISMS-PF the default data format is VTK and there is no reason to seek an
alternative. If an alternative file format is required then the researcher
could code a C++ function to write data in an alternative format to VTK such as
NetCDF.

As a general rule it is best to choose file formats that work with the tools
already in use and / or that your colleagues are using. There are other
considerations to be aware of though. Human readable formats such as CSV and
JSON are often useful for small medium data sets (such as derived quantities)
as some metadata can be embedded alongside the raw data resulting in a FAIRer
data product than standard binary formats. Some binary file formats also
support metadata and might be more useful for final data curation of a
phase-field study even if not used during the research process. One main
benefit of using binary data (beyond saving disk space) is the ability to
preserve full precision for floating point numbers. The longevity of file
formats should be considered as well. A particularly egregious case of ignoring
longevity would be using the Pickle file format in Python, which is both
language dependent and code dependent. It is an example of data serialization,
which is used mainly for in-process data storage for asynchronous tasks, but
not good for long term data storage.

There are many binary formats used for storing field data based on an Eulerian
mesh or grid. Common formats for field data are NetCDF, VTK, XDMF and
EXODUS. Within the phase-field community, VTK seems to be the mostly widely
used. VTK is actually a visualization library, but supports a number of
different native file formats based on both XML and HDF5 (both non-binary and
binary). The VTK library works well with FE simulations supporting many
different element types as well as parallel data storage for domain
decomposition.  See the [XML file formats documentation][vtk-xml] for VTK for
an overview of zoo of different file extensions and their meaning. In contrast
to VTK, NetCDF is more geared towards gridded data having arisen from
atmospheric research, which uses more FD and FV than FE. For a comparison of
performance and metrics for different file types see the
[Python MeshIO tools's README.md][meshio].

The Python MeshIO tool is a good place to start for IO when writing custom
phase-field codes in Python (or Julia using `pyimport`). MeshIO is also a good
place to start for exploring, debugging or picking apart file data in an
interactive Python environment, which can be harder to do with dedicated
viewing tools like Paraview. The scientific Python ecosystem is very rich with
tools for data manipulation and storage such as Pandas, which supports storage
in many different formats, and xarray for higher dimensional data. xarray
supports NetCDF file storage, which includes coordinate systems and metadata in
HDF5. Both Pandas and xarray can be used in a parallel or a distributed manner
in conjucntion with Dask. Dask along with xarray supports writing to the Zarr
data format. Zarr allows data to be stored on disk during analysis to avoid
loading the entire data object into memory.

- https://aaltoscicomp.github.io/python-for-scicomp/work-with-data/
- https://docs.vtk.org/en/latest/index.html
- https://docs.xarray.dev/en/stable/user-guide/io.html=

(label-restarts)=
### Recovering from crashes and restarts

A study from 2020 of HPC systems calculated the success rate (I.e. no error
code on completion) of multi-node jobs with non-shared memory at between 60%
and 70% {cite}`Kumar2020`. This success rate diminishes rapidly as the run time
of jobs increases. Needless to say that check-pointing is absolutely required
for any jobs of more than a few hours. Nearly everyday, an HPC platform will
experience some sort of failure {cite}`Benoit2022b`, {cite}`Aupy2014`. That
doesn't mean that every job will fail every day, but it would be optimistic to
think that jobs will go beyond a week without some issues. Given that fact one
can estimate how long it might take to run a job without check-pointing. A very
rough estimate for expected completion time assuming instantaneous restarts and
no queuing time is given by,

$$ E(T) = \frac{1}{2} \left(1 + e^{T / \mu} \right) T $$

where $T$ is the nominal job completion time with no failures and
$\mu$ is the mean time to failure. The formula predicts an expected
time of 3.8 days for a job that nominally runs for 3 days with a $\mu$
of one week. The formula is of course a gross simplification and
includes many invalid assumptions, but regardless of the assumed
failure distribution the exponential time increase without
check-pointing is inescapable. Assuming that we're agreed on the need
for checkpoint, the next step is to decide on the optimal time
interval between checkpoints. This is given by the well known
Young/Daly formula,

$$ W = \sqrt{2 \mu C} $$

where $C$ is the time taken for a checkpoint {cite}`Benoit2022a`,
{cite}`BautistaGomez2024`. The Young/Daly formula accounts for the trade off
between the start up time cost for a job to get back to its original point of
failure and the cost associated with writing the checkpoint to disk. For
example, with a weekly failure rate and $C=6$ minutes, $W=5.8$ hours. In
practice these estimates for $\mu$ and $C$ might be a little pessimistic, but
be aware of the trade off {cite}`Benoit2022b`. Note that some HPC systems have
upper bounds on run times (e.g. TACC has a 7 days time limit so $\mu<7$ days
regardless of other system failures).

Given the above theory, what is the some practical advice for
check-pointing jobs?

- Estimate both $\mu$ and $C$. It might be worth discussing the $\mu$
  value with the HPC cluster administrator to get some valid
  numbers. Of course $C$ can be estimated by running test jobs. It's
  good to know if you should be writing checkpoints every day or every
  hour -- definitely not every minute!
- Ensure that restarts are deterministic (i.e. results don't change
  between a job that restarts and one that doesn't). One way to do
  this is to hash output files assuming that the simulation itself is
  deterministic.
- Consider using a checkpointing library if you're using a custom
  phase-field code or even a workflow tool such as Snakemake which has
  the inbuilt ability to handle checkpointing. A tool like Snakemake
  is good for large parameter studies where it is difficult to keep
  track of which jobs wrote which files. The `pickle` library is
  acceptable for checkpointint Python programs _in this short-lived
  circumstance_.
- Use the built-in checkpointing available in the phase-field code
  that you're using.
- Whatever system is being used, check that the checkpointing machinery
  actually works and is deterministic.

Some links to further reading:

- <https://hivehpc.haifa.ac.il/index.php/slurm?start=5>
- <https://icl.utk.edu/files/publications/2022/icl-utk-1569-2022.pdf>
- <https://inria.hal.science/hal-03264047/file/rr9413.pdf>
- <https://www.sciencedirect.com/science/article/abs/pii/S0743731513002219>
- <https://icl.utk.edu/files/publications/2022/icl-utk-1569-2022.pdf>
- <https://www.ittc.ku.edu/~sun/publications/fgcs24.pdf>
- <https://www.ittc.ku.edu/~sun/publications/fgcs24.pdf>
- <https://icl.utk.edu/files/publications/2020/icl-utk-1385-2020.pdf>
- <https://ftp.cs.toronto.edu/csrg-technical-reports/621/ut-csrg-621.pdf>
- <https://arxiv.org/pdf/2012.00825>
- <https://icl.utk.edu/~herault/papers/007%20-%20Checkpointing%20Strategies%20for%20Shared%20High-Performance%20Computing%20Platforms%20-%20IJNC%20(2019).pdf>
- <https://dl.acm.org/doi/10.1145/2184512.2184574>
- <https://engineering.purdue.edu/dcsl/publications/papers/2020/fresco_dsn20_cameraready.pdf>
- [Job failures](https://pdf.sciencedirectassets.com/271503/1-s2.0-S0898122111X00251/1-s2.0-S0898122111005980/main.pdf)
- <https://www.cs.cmu.edu/~bianca/dsn06.pdf>

### Using Workflow Tools

The authors of this article use Snakemake for their workflows so will
discuss this in particular, but most of the ideas will apply to other
workflow tools. In general when running many phase-field jobs for a
parameter study or dealing with many pre and post-processing steps, it
is wise to employ a workflow tool such as Snakemake. One of the main
benefits of workflow tools is the automation of all the steps in a
workflow that researchers often neglect to implement in the absence of
a workflow tool (e.g. with bash scripts). This forces a structure and
the researchers to think carefully about the inputs / outputs and task
graph. As a side effect, the graph structure produces a much FAIRer
research object when the research is published and shared and even so
that the researcher can rerun the simulation steps in the future. For
example, when using Snakemake, the `Snakefile` itself is a clear
record of the steps required to re-execute the workflow. Ideally, the
`Snakefile` will include all the steps required to go from the raw
inputs to images and data tables used in publications, but this might
not always be possible.

A secondary impact of using a workflow tool is that it often imposes a
directory and file structure on the project. For example, Snakemake
has an ideal suggested structure. An example folder structure when
using Snakemake would look like the following.

```plain
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

Notice that the above directory strucuture includes the `envs`
directory. This allows diffferent steps in the workflow to be run in
diffferent types of environments. The benefit of this is that the
steps can be highly hetrogeneous in terms of the required
computational enviornment. Additionally, most workflow tools will
support both HPC and local workstation execution and make porting
between systems easier.

See {cite}`Moelder2021` for a more detailed overview of Snakemake and a list of
other good workflow tools.

- https://snakemake.readthedocs.io/en/stable/snakefiles/deployment.html

(label-hpc-environments)=
### HPC Environments and parallel writes

## Data Curation

Data curation involves the steps required to turn an unstructured data
from a research project into a coherent research data object
satisfying the principles of FAIR data. A robust data curation process
is often a requirement for compliance for funding requirements and to
simply meet the most basic needs of transparency in scientific
research. The main benefits of data curation include (see
[DCC](https://www.dcc.ac.uk/guidance/how-guides/develop-data-plan#Why%20develop))

_To Do:_ Simulation FAIR data paragraph and importance of metadata

The fundamental steps to curate a computational research project into
a research data object and publish are as follows.

- Automate the entire computational workflow where possible during the
  research process from initial inputs to final research products such
  as images and data tables.
- Publish the code and workflows appropriately during development (see
  the ... guide).
- Employ a suitable metadata standard where possible to describe
  different aspects of the research project such as the raw data
  files, derived data assets, software environments, numerical
  algorithms and problems specification.
- Identify the significant raw and derived data assets that are
  required to produce the final research products.
- License the research work appropriately. This may require a separate
  license for the data products as they are generally not archived in
  the code repository.
- Select a data repository to curate the data
- Obtain a DOI for the data object and link with other research
  products

The above steps are difficult to implement near the conclusion of a research
project. The authors suggest implementing the majority of these steps at the
outset of the project and developing these steps as part of a protocol for all
research projects within a computational materials research group.

### Automation

Automating workflows in computational materials science is useful for many
reasons, however, for data curation purposed it provides and added benefit. In
short, an outlined workflow associated with a curated FAIR object is a major
way to improve FAIR quality for subsequent researchers. For most workflow
tools, the operation script outlining the workflow graph is the ultimate form
of metadata about how the archived data files are used or generated during the
research. For example, with Snakemake, the `Snakefile` has clearly outlined
inputs and outputs as well as the procedure associated with each input / output
pair. In particular, the computational environment, command line arguments,
environment variables are recorded as well as the order of execution for each
step.

In recent years there have been efforts in the life sciences to provide a
minimum workflow for independent code execution during the peer review
process. The CODECHECK initiative {cite}`Nuest2021` tries to provide a standard
for executing workflows and a certification if the workflow satisifies basic
criteria. These types of efforts will likely be used within the compuational
materials science community in the coming years so adopting automated workflow
tools as part of your research will greatly benefit this process.

- See also {cite}`Leipzig2021`

### Metadata Standards

### Publish the codes and workflows during development

### Identifying the significant data assets

### Licensing

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
[mehio]: https://github.com/nschloe/meshio?tab=readme-ov-file#performance-comparison
[vtk-xml]: https://docs.vtk.org/en/latest/design_documents/VTKFileFormats.html#xml-file-formats

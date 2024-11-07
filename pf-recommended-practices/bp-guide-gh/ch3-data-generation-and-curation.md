# Data Generation and Curation

- *[Trevor Keller](https://www.nist.gov/people/trevor-keller), NIST*, [@tkphd]
- *[Daniel Wheeler](https://www.nist.gov/people/daniel-wheeler), NIST*, [@wd15]
- *[Damien Pinto](https://ca.linkedin.com/in/damien-pinto-4748387b), McGill*, [@DamienPinto]

## Overview

- Look at lit on data and see how this is implemented
- Generation and dissemination

## Ideas

- Data formats
- FAIR
- Metadata (hierarchical data standards (look for current versions))
- What standards exist
- One or two examples of phase field data stored currently
  - Use an existing
  - Create our own example
- Practical choices for storing data (figshare, zenodo, dryad, MDF)
- Deciding what data to keep
  - What data to store when publishing
  - What is supplementary material versus store versus leave on hard drive
- Lit review, good citations
- minting DOIs for the data
- might include simulatioin execution and logging
  - how frequently to store data
  - how to store

## How do we want to structure the sections?

1. Intro (Daniel)
   - What is data?
   - What is metadata?
   - Why do we need curate data?
   - What is the motivation for this document?
     - What should the reader get out of this document
     - Why is this useful
   - Create a distinction between software, data, and post-processed results

2. Data Generation (Trevor)
   - HPC
   - file systems
   - data formats
     - formats not to use (e.g., don't use serialization that depends on the version of the code that reads and writes because code changes)
     - don't use pickles
   - restarts
   - data frequency
   - post-processing -> refer to other document for this
   - precision
   - importance of folder structure

3. Data Curation (Trevor)
   - Why is curating data important?
   - Why do we need to store our data
   - What formats can we use
   - Is my data too large, data sizes
   - What is useful for third party / subsequent users
   - How might the data be used for AI or something else
   - Could a reviewer make use of your curated data
   - Storing post-processed data and raw data and which to store or keep
   - Minting DOIs for your software when publishing a paper
   - FAIR

4. Metadata standards (Daniel)
   - Why do we need to keep some metadata beyond the data
   - Zoo of things like data dictionaries, ontologies
     - however, these are not well developed for our use case
   - For example, you curate on Zenodo
   - what extra data should you include
     - how to describe the data files
     - how to maintain some minimalist info about the simulation that generated the run
     - When, why and how I ran this simulation
     - What software
     - Give example of a yaml file with 10 flat fields
   - The future should be better in this regard. People actively working to improve this issue.

5. Examples
   - Practical examples (Trevor)
   - Using Zenodo for a PFHub record to store data and metadata
     - Relatively rich metadata scheme

   - Simulation from scratch (Damien)
     - data generation
       - folder structure
       - HPC issues with data
       - capture process / descriptive parameters for the data that
         are useful for subsequent ML practitioners that use the data
     - ML / store data
     - Narrative of what gets stored to disk
     - Decisions of what to keep and how frequently to save data
     - Auxiliary metadata decisions

 6. Summary (Daniel)
 7. Biblio (Daniel)

---

## Old version

- Save the data from your published work as much as possible, with meta data
- Save the inputs used to produce the results from all your published work

### FAIR Data

We discussed the [FAIR Principles] at [CHiMaD Phase-Field XIII][fair-phase-field]:

#### Findable

- [ ] (Meta)data are assigned a globally unique and persistent identifier
- [ ] Data are described with rich metadata (defined by R1 below)
- [ ] Metadata clearly and explicitly include the identifier of the data they describe
- [ ] (Meta)data are registered or indexed in a searchable resource

#### Accessible

- [ ] (Meta)data are retrievable by their identifier using a standardized
   communications protocol
   - [ ] The protocol is open, free, and universally implementable
   - [ ] The protocol allows for an authentication and authorisation procedure,
         where necessary
- [ ] Metadata are accessible, even when the data are no longer available

#### Interoperable

- [ ] (Meta)data use a formal, accessible, shared, and broadly applicable language
      for knowledge representation.
- [ ] (Meta)data use vocabularies that follow FAIR principles
- [ ] (Meta)data include qualified references to other (meta)data

#### Reusable

- [ ] (Meta)data are richly described with a plurality of accurate and relevant attributes
   - [ ] (Meta)data are released with a clear and accessible data usage license
   - [ ] (Meta)data are associated with detailed provenance
   - [ ] (Meta)data meet domain-relevant community standards

### Zenodo

Historically, [PFHub] has accepted datasets linked from any host on the Web.
At this time, we recommend using [Zenodo] to host your benchmark data. Why? *It's not "just" a shared folder.*

* Guided prompts to describe what you're uploading
* DOI is automatically assigned to your dataset
* Basic metadata exported in multiple formats
* Browser-based viewers for CSV, Markdown, PDF, images, videos

#### Metadata Examples

Zenodo gives you the option to import a repository directly from GitHub. The original [FAIR Phase-field talk](https://doi.org/10.5281/zenodo.6540105) was "uploaded" this way, producing the following record. While basic authorship information was captured, this tells an interested person or machine nothing meaningful about the dataset.

```json
{
  "@context": "https://schema.org/",
  "@id": "https://doi.org/10.5281/zenodo.6540105",
  "@type": "SoftwareSourceCode",
  "name": "tkphd/fair-phase-field-data: CHiMaD Phase-field XIII",
  "description": "FAIR Principles for Phase-Field Practitioners",
  "version": "v0.1.0",
  "license": "",
  "identifier": "https://doi.org/10.5281/zenodo.6540105",
  "url": "https://zenodo.org/record/6540105",
  "datePublished": "2022-05-11",
  "creator": [{
      "@type": "Person",
      "givenName": "Trevor",
      "familyName":  "Keller",
      "affiliation": "NIST"}],
  "codeRepository": "https://github.com/tkphd/fair-phase-field-data/tree/v0.1.0"
}
```

The *strongly* preferred method is to upload files directly. The following record represents an upload for [Benchmark 1b using HiPerC](https://doi.org/10.5281/zenodo.1124941). I would consider this metadata ***rich!***

```json
{
  "@context": "https://schema.org/",
  "@id": "https://doi.org/10.5281/zenodo.1124941",
  "@type": "Dataset",
  "name": "hiperc-gpu-cuda-spinodal"
  "description": "Solution to the CHiMaD Phase Field benchmark problem on spinodal decomposition using CUDA, with a 9-point discrete Laplacian stencil",
  "identifier": "https://doi.org/10.5281/zenodo.1124941",
  "license": "https://creativecommons.org/licenses/by/4.0/legalcode",
  "url": "https://zenodo.org/record/1124941",
  "datePublished": "2017-12-21",
  "creator": [{
      "@type": "Person",
      "@id": "https://orcid.org/0000-0002-2920-8302",
      "givenName": "Trevor",
      "familyName":  "Keller",
      "affiliation": "NIST"}],
  "keywords": ["phase-field", "pfhub", "chimad"],
  "sameAs": ["https://doi.org/10.6084/m9.figshare.5715103.v2"],
  "distribution": [
    {
      "@type": "DataDownload",
      "contentUrl": "https://zenodo.org/api/files/ce1ca4a3-b6bc-4e2c-9b70-8fe45fc243fd/free-energy-9pt.csv",
      "encodingFormat": "csv"
    }, {
      "@type": "DataDownload",
      "contentUrl": "https://zenodo.org/api/files/ce1ca4a3-b6bc-4e2c-9b70-8fe45fc243fd/spinodal.0000000.png",
      "encodingFormat": "png"
    }, {
      "@type": "DataDownload",
      "contentUrl": "https://zenodo.org/api/files/ce1ca4a3-b6bc-4e2c-9b70-8fe45fc243fd/spinodal.0100000.png",
      "encodingFormat": "png"
    }, {
      "@type": "DataDownload",
      "contentUrl": "https://zenodo.org/api/files/ce1ca4a3-b6bc-4e2c-9b70-8fe45fc243fd/spinodal.0200000.png",
      "encodingFormat": "png"
    }]
}
```

#### Metadata Files

After uploading the HiPerC simulation data, I also registered it with PFHub using `meta.yaml`. This file tells the website-generating machinery what to do with the dataset, and provides additional information about the resources required to perform the simulation.

```yaml
---
benchmark:
  id: 1b
  version: '1'
data:
- name: run_time
  values:
  - sim_time: '200000'
    wall_time: '7464'
- name: memory_usage
  values:
  - unit: KB
    value: '308224'
- description: free energy data
  url: https://zenodo.org/api/files/ce1ca4a3-b6bc-4e2c-9b70-8fe45fc243fd/free-energy-9pt.csv
- description: microstructure at t=0
  type: image
  url: https://zenodo.org/api/files/ce1ca4a3-b6bc-4e2c-9b70-8fe45fc243fd/spinodal-000000.png
- description: microstructure at t=100,000
  type: image
  url: https://zenodo.org/api/files/ce1ca4a3-b6bc-4e2c-9b70-8fe45fc243fd/spinodal-100000.png
- description: microstructure at t=200,000
  type: image
  url: https://zenodo.org/api/files/ce1ca4a3-b6bc-4e2c-9b70-8fe45fc243fd/spinodal-200000.png
metadata:
  author:
    email: trevor.keller@nist.gov
    first: Trevor
    github_id: tkphd
    last: Keller
  hardware:
    acc_architecture: gpu
    clock_rate: '1.48'
    cores: '1792'
    cpu_architecture: x86_64
    nodes: 1
    parallel_model: threaded
  implementation:
    repo:
      url: https://github.com/usnistgov/hiperc
      version: b25b14acda7c5aef565cdbcfc88f2df3412dcc46
  simulation_name: hiperc_cuda
  summary: HiPerC spinodal decomposition result using CUDA on a Tesla P100
  timestamp: 18 December, 2017
```

This file is not part of my dataset: it resides in the [PFHub repository on GitHub]. Furthermore, since the structure of this file specifically suits PFHub, it is of no use at all to other software, websites, or researchers.

### Structured Data Schemas

In the Zenodo metadata above, note the `@context` fields: [Schema.org] is a [structured data schema] *and controlled vocabulary* for describing things on the Internet. How is this useful?

Consider the [CodeMeta] project. It creates metadata files for software projects using [Schema.org] building blocks. There's even a handy [CodeMeta Generator]! If you maintain a phase-field software framework, you can (and should!) use it to document your code in a standards-compliant, machine-readable format. This improves interoperability and reusability!

```json
{
    "@context": "https://doi.org/10.5063/schema/codemeta-2.0",
    "@type": "SoftwareSourceCode",
    "license": "https://spdx.org/licenses/CC-PDDC",
    "codeRepository": "git+https://github.com/usnistgov/hiperc",
    "dateCreated": "2017-08-07",
    "dateModified": "2019-03-04",
    "downloadUrl": "https://github.com/usnistgov/hiperc/releases/tag/v1.0",
    "issueTracker": "https://github.com/usnistgov/hiperc/issues",
    "name": "HiPerC",
    "version": "1.0.0",
    "description": "High-Performance Computing in C and CUDA",
    "applicationCategory": "phase-field",
    "developmentStatus": "inactive",
    "programmingLanguage": ["C", "CUDA", "OpenCL", "OpenMP", "TBB"],
    "author": [
        {
            "@type": "Person",
            "@id": "https://orcid.org/my-orcid?orcid=0000-0002-2920-8302",
            "givenName": "Trevor",
            "familyName": "Keller",
            "email": "trevor.keller@nist.gov",
            "affiliation": {
                "@type": "Organization",
                "name": "NIST"
            }
        }
    ]
}
```

That's nice! But what about our datasets? Shouldn't the [PFHub] metadata "describing" a dataset live alongside that data?

#### Towards a Phase-Field Schema

We are working to build a phase-field schema (or schemas) using [Schema.org] and the [schemaorg] Python library. The work-alike port of `meta.yaml` looks like the following.

> *N.B.:* We're going to deploy a generator similar to CodeMeta's so you won't have to write this!

```json
{
    "@context": "https://www.schema.org",
    "@type": "DataCatalog",
    "author": [
        {
            "@type": "Person",
            "affiliation": {
                "@type": "GovernmentOrganization",
                "name": "Materials Science and Engineering Division",
                "parentOrganization": {
                    "@type": "GovernmentOrganization",
                    "name": "Material Measurement Laboratory",
                    "parentOrganization": {
                        "@type": "GovernmentOrganization",
                        "address": {
                            "@type": "PostalAddress",
                            "addressCountry": "US",
                            "addressLocality": "Gaithersburg",
                            "addressRegion": "Maryland",
                            "postalCode": "20899",
                            "streetAddress": "100 Bureau Drive"
                        },
                        "identifier": "NIST",
                        "name": "National Institute of Standards and Technology",
                        "parentOrganization": "U.S. Department of Commerce",
                        "url": "https://www.nist.gov"
                    }
                }
            },
            "email": "trevor.keller@nist.gov",
            "familyName": "Keller",
            "givenName": "Trevor",
            "identifier": "tkphd",
            "sameAs": "https://orcid.org/0000-0002-2920-8302"
        }, {
            "@type": "Person",
            "affiliation": {
                "@type": "GovernmentOrganization",
                "name": "Materials Science and Engineering Division",
                "parentOrganization": {
                    "@type": "GovernmentOrganization",
                    "name": "Material Measurement Laboratory",
                    "parentOrganization": {
                        "@type": "GovernmentOrganization",
                        "address": {
                            "@type": "PostalAddress",
                            "addressCountry": "US",
                            "addressLocality": "Gaithersburg",
                            "addressRegion": "Maryland",
                            "postalCode": "20899",
                            "streetAddress": "100 Bureau Drive"
                        },
                        "identifier": "NIST",
                        "name": "National Institute of Standards and Technology",
                        "parentOrganization": "U.S. Department of Commerce",
                        "url": "https://www.nist.gov"
                    }
                }
            },
            "email": "daniel.wheeler@nist.gov",
            "familyName": "Wheeler",
            "givenName": "Daniel",
            "identifier": "wd15",
            "sameAs": "https://orcid.org/0000-0002-2653-7418"
        }
    ],
    "dataset": [
        {
            "@type": "Dataset",
            "distribution": [
                {
                    "@type": "PropertyValue",
                    "name": "parallel_nodes",
                    "value": 1
                }, {
                    "@type": "PropertyValue",
                    "name": "cpu_architecture",
                    "value": "amd64"
                }, {
                    "@type": "PropertyValue",
                    "name": "parallel_cores",
                    "value": 12
                }, {
                    "@type": "PropertyValue",
                    "name": "parallel_gpus",
                    "value": 1
                }, {
                    "@type": "PropertyValue",
                    "name": "gpu_architecture",
                    "value": "nvidia"
                }, {
                    "@type": "PropertyValue",
                    "name": "gpu_cores",
                    "value": 6144
                }, {
                    "@type": "PropertyValue",
                    "name": "wall_time",
                    "unitCode": "SEC",
                    "unitText": "s",
                    "value": 384
                }, {
                    "@type": "PropertyValue",
                    "name": "memory_usage",
                    "unitCode": "E63",
                    "unitText": "mebibyte",
                    "value": 1835
                }
            ],
            "name": "irl"
        }, {
            "@type": "Dataset",
            "distribution": [
                {
                    "@type": "DataDownload",
                    "contentUrl": "8a/free_energy_1.csv",
                    "name": "free energy"
                }, {
                    "@type": "DataDownload",
                    "contentUrl": "8a/solid_fraction_1.csv",
                    "name": "solid fraction"
                }, {
                    "@type": "DataDownload",
                    "contentUrl": "8a/free_energy_2.csv",
                    "name": "free energy"
                }, {
                    "@type": "DataDownload",
                    "contentUrl": "8a/solid_fraction_2.csv",
                    "name": "solid fraction"
                }, {
                    "@type": "DataDownload",
                    "contentUrl": "8a/free_energy_3.csv",
                    "name": "free energy"
                }, {
                    "@type": "DataDownload",
                    "contentUrl": "8a/solid_fraction_3.csv",
                    "name": "solid fraction"
                }
            ],
            "name": "output"
        }
    ],
    "dateCreated": "2022-10-25T19:25:02+00:00",
    "description": "A fake dataset for Benchmark 8a unprepared using FiPy by @tkphd & @wd15",
    "isBasedOn": {
        "@type": "SoftwareSourceCode",
        "codeRepository": "https://github.com/tkphd/fake-pfhub-bm8a",
        "description": "Fake benchmark 8a upload with FiPy",
        "runtimePlatform": "fipy",
        "targetProduct": "amd64",
        "version": "9df6603e"
    },
    "isPartOf": {
        "@type": "Series",
        "identifier": "8a",
        "name": "Homogeneous Nucleation",
        "url": "https://pages.nist.gov/pfhub/benchmarks/benchmark8.ipynb"
    },
    "keywords": [
        "phase-field",
        "benchmarks",
        "pfhub",
        "fipy",
        "homogeneous-nucleation"
    ],
    "license": "https://www.nist.gov/open/license#software"
}
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

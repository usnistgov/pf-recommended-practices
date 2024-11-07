# Result Dissemination

Mike Tonks, Damien Tourret, Katsuyo Thornton, Jon Guyer

How you present and disseminate your results is another critical aspect of applying the phase-field method. Here, we provide recommended practices on disseminating your results.

## Recommended Practices for Postprocessing and Plotting:

### Present dimensional results whenever possible

The phase-field method is inherently dimensional, meaning that each of its model parameters have specific dimensions in terms of length, time, energy, etc. These values vary for different materials and systems, based on material properties such as free energies, diffusivities, interfacial energies, and so on. However, it is a common practice to nondimensionalize a model to simplify the numerical solution. Nondimensionalization has also been used to avoid the need to have accurate material properties for a specific material system. In those cases, results are qualitative at best and are not material specific.

It is recommended to present material-specific results using dimensional properties. Therefore, if you nondimensionalize your model, **always** dimensionalize your results before plotting if at all possible. Quantitative, material specific results are always higher impact than qualitative results.

Presenting a dimensionless model with dimensional properties should be used as a sanity check that the dimensions and parameters are consistent. For that matter, even dimensional models are usually coded in a way that does not actually incorporate units in a self-consistent way, so careful balancing of equations is important. If your simulation framework supports dimensional units, you should use them.

### Make plots and figures that clearly illustrate your data

Well-constructed figures can greatly enhance the presentation of your data. There many general guidelines and recommendations that should be followed. Rather that reproduce them all here, we link to several valuable resources that provide great recommendations:
* Video 7: Incorporating Illustrations from [Writing as an Engineer or Scientist]( https://sites.psu.edu/scientificwriting/tutorial-reports/).
* [Nature reference](https://www.nature.com/documents/natrev-artworkguide_PS.pdf) about making good figures with excellent suggestions.
* [A Brief Guide to Designing Effective Figures for the Scientific Paper](https://onlinelibrary.wiley.com/doi/full/10.1002/adma.201102518).

Figures can be broadly divided into two main categories, visualization of the actual simulation results and plots of postprocessed data extracted from the results. 
To create images of phase field simulation results, you need post-processing visualization software. There are a number of powerful tools for visualizing simulation results, including:
* [ParaView - open source]( https://www.paraview.org/)
* [PyVista - open source]( https://pyvista.org/)
* [VisIt - open source]( https://visit-dav.github.io/visit-website/index.html)
* [Origin – commercial]( https://www.originlab.com/origin)

These tools read in common data formats, and so your code needs to be able to output results in a supported format. [VTK formats]( https://docs.vtk.org/en/latest/design_documents/VTKFileFormats.html) are supported by all of these tools.

To create plots of postprocessed data, the data is typically outputted directly from your simulation tool or from visualization software in formats like “.csv”. You need tools that can read in these data and then generate various types of plots. We recommend tools that allow scripts (as we discuss more, below); such tools include:
* Python, using [matplotlib](https://matplotlib.org/) for plotting and [pandas](https://pandas.pydata.org/) for csv reading. [Seaborn](https://seaborn.pydata.org/) is a useful tool as well.
* [MATLAB](https://www.mathworks.com/help/matlab/creating_plots/types-of-matlab-plots.html)
* [GNUPlot](http://www.gnuplot.info/)


You should also carefully consider the color scheme used in your plots. For example, some color schemes work for color blind viewers while others do not. For more information about choosing a color scheme for your figure, see Fig. 6 from [The misuse of colour in science communication](https://www.nature.com/articles/s41467-020-19160-7).

### Automate postprocessing and plot generation

When preparing results for dissemination, whether for a presentation, report, or paper, you will often find things that need to be changed and therefore must regenerate figures and plots. This is a normal part of research. However, the work required to regenerate figures and plots can be significantly reduced using automated scripts or a well-defined and documented reproducible protocol. The goal is to reduce the amount of "manual" operations to a minimum. Consider a paper you wrote a year or more ago; how readily could you regenerate its figures?

Creating an image that is clear and provides all of the information needed for a typical publication in vizualization tools can often be a time-consuming process. Luckily, they often have ways of saving a given configuration that can then be reloaded to quickly generate a similar image. In Paraview, this is done by saving and loading the state.

For plots of postprocessed, data is often outputted from research codes and then plotted using software. Plots can be generated and then edited manually or scripts can be used that define all aspects of the plot. The use of scripts may make the time it takes to create the plot the first time slightly longer but will reduce the time to regenerate the plots to almost zero. Scripts also ensure that your plots have a consistent look when regenerated. 

## Recommended Practices for Papers

### Provide all of the information needed for someone else to reproduce your results

A critical aspect of scientific publication is the reproduction of results by other researchers. For papers using the phase-field method, that means you need to provide enough information that the reader could reproduce your results. It can be helpful to put yourself in the shoes of your readers; if you come back to this work in a few years, how readily could you perform these simulations again? This information does not have to be provided in the main body of the paper, but can be in appendices or supplemental information. The provided information should include:

* The value used for every parameter in the model. Even if the values are given in one of your references, you should still provide the values in your paper. 
* The domain dimension and size
* Initial conditions and boundary conditions
* Other conditions used for each simulation, including temperature and length of time.
* The discretization scheme, including types of spatial and temporal discretization and spatial discretization spacing and time step size.

Even if using a proprietary simulation tool, the input files should be published as the ultimate source of how a simulation was performed. There are potentially multiple levels that may impact reproducibility:

1. The parameter file(s), configuration file(s), or driver script(s) for the specific simulation.
2. The environment variables and batch script(s) used to initiate a series of simulations.
3. The HPC scheduler configuration file. 

GUI configuration is particularly challenging from a reproducibility standpoint, but even command-line arguments that affect the simulation are undesirable unless they are captured (and published) in the next highest level of configuration/script/driver. **[note: Some of this may better belong in, or cross-reference to, [Software-Development](ch4-software-development.md).]**

### Clearly state all assumptions

All models are generated by making assumptions and approximations. You should state every assumption and approximation made in the formulation of the model and why they were made. This includes any assumptions involved in equations you have taken from the literature. It is impossible for a reader to understand the strengths and weaknesses of your model without knowing the assumptions and approximations.

### Include key verification results 

It is natural to present large simulation results that capture complex behaviors that occur in nature. However, it can often be difficult to assess how well a model is performing in such large simulation results because the behavior is so complex. Therefore, when presenting phase-field model/simulation results, it is important to validate/verify the accuracy of the model and its implementation by simulating simple problems that more clearly demonstrate the accuracy of the model/code. Such simple problems may have known analytical solutions, against which the simulated results can be compared for verification (see [Code Verification](ch4-software-development.md#code-verification)]). Key results from such studies should be shared along with the main simulation results in a repository or in a supplementary document of the associated publication.

### Share your input file, output data, and/or codes 

The best practice is to share key output data and input files where it is findable and accessible in a manner that the data is interoperable and reusable (FAIR); see [Data-Generation-and-Curation](ch3-data-generation-and-curation.md). For projects funded by federal agencies, your data management plan may also require you to publish your data associated with your publication. There are several public repositories to which your data/files can be uploaded and shared, which are described in [Data-Generation-and-Curation](ch3-data-generation-and-curation.md). The data should be accompanied by key metadata, including information about the software used to generate the output and the associated publication. The publication should also be associated with the shared data via DOI. If the codes are not publicly available, it is also encouraged that they are shared as well.

## About
 [![License](https://img.shields.io/pypi/l/jax-cosmo)](https://github.com/EdwardBerman/shopt/blob/main/LICENSE) [![All Contributors](https://img.shields.io/badge/all_contributors-2-orange.svg?style=flat-square)](#contributors)

**Shear Optimization** with **ShOpt.jl**, a julia library for empirical point spread function characterizations. We aim to improve upon the current state of Point Spread Function Modeling by using Julia to leverage performance gains, use a different mathematical formulation than the literature to provide more robust analytic and pixel grid fits, improve the diagnostic plots, and add features such as wavelets and shapelets. At this projects conclusion we will compare to existing software such as PIFF and PSFex. Work done under [McCleary's Group](https://github.com/mcclearyj).

### Analytic Profile Fits 
We adopt the following procedure to ensure our gradient steps never take us outside of our constraints

![image](READMEassets/reparameterization.png)

### Pixel Grid Fits                                                                                                        
For doing Pixel Grid Fits we use an autoencoder model to reconstruct the Star                                              
![image](READMEassets/nn.png) 

### Interpolation Across the Field of View
[s, g1, g2] are all interpolated across the field of view. Each Pixel is also given an interpolation across the field of view for an nth degree polynomial in (u,v), where n is supplied by the user

## Inputs and Outputs
Currently, the inputs are JWST Point Spread Functions source catalogs. The current outputs are images of these Point Spread Functions, Learned Analytic Fits, Learned Pixel Grid Fits, Residual Maps, Loss versus iteration charts, and p-value statisitcs. Not all functionality is working in its current state. Planned functionality for more Shear checkplots.

### Inputs 

| Image                             | Description                        |
|-----------------------------------|------------------------------------|
| ![image](READMEassets/input.png)  | Star Taken From Input Catalog      |


### Outputs

| Image                                              | Description                                                                                                                         |
|----------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| summary.shopt                                      | Fits File containing summary statistics and information to reconstruct the PSF                                                      |
| shopt.yml                                          | Config File for Tunable Parameters Such as Stamp Size, Stopping Gradients, Plotting Options, etc.                                   |
| ![image](READMEassets/pgf9.png)                    | Pixel Grid Fit for the Star Above                                                                                                   |
| ![image](READMEassets/hmresid.png)                 | Residual Map for Above Model and Fit                                                                                                |
| ![image](READMEassets/s_uv.png)                    | s varying across the field of view                                                                                                  |
| ![image](READMEassets/g1_uv.png)                   | g1 varying across the field of view                                                                                                 |
| ![image](READMEassets/g2_uv.png)                   | g2 varying across the field of view                                                                                                 |
| ![image](READMEassets/parametersHistogram.png)     | Histogram for learned profiles for each star in an analytic fit with their residuals                                                |
| ![image](READMEassets/parametersScatterplot.png)   | Same data recorded as a scatterplot with and without outliers removed and with error bars                                           |

NB: This is not a comprehensive list, only a few cechkplots are presented. See the shopt.yml to configure which plots you want to see and save!

## Running
### Command 
To run `shopt.jl`

First use Source Extractor to create a catalog for ShOpt to accept and save this catalog in the appropriate directory

Run ```julia shopt.jl [configdir] [outdir] [catalog] [scifile]```

There is also a shell script that runs this command so that the user may call shopt from a larger program they are running

### Dependencies 
Not all of these will be strictly necessary depending on the checkplots you produce, but for full functionality of ShOpt the following are necessary. Source Extractor (or Source Extractor ++) is also not a strict dependency, but in practice one will inevitably install to generate a catalog.

| Julia            | Python     | Binaries | Julia          | Julia          |
|------------------|------------|----------|----------------|----------------|
| Plots            | matplotlib | SEx      | ProgressBars   | BenchmarkTools |
| ForwardDiff      | astropy    |          | UnicodePlots   | Measures       |
| LinearAlgebra    | numpy      |          | CSV            | Dates          |
| Random           |            |          | FFTW           | YAML           |
| Distributions    |            |          | Images         | CairoMakie     |
| SpecialFunctions |            |          | ImageFiltering | Flux           |
| Optim            |            |          | DataFrames     | QuadGK         |
| IterativeSolvers |            |          | PyCall         | Statistics     |

### Set Up 
Start by cloning this repository. There are future plans to release ShOpt onto a julia package repository, but for now the user needs these files contents.

The dependencies can be installed in the Julia REPL. For example:
```julia
import Pkg; Pkg.add("PyCall")
```

We also provide dependencies.jl, which you can run to download all of the Julia libraries automatically by reading in the imports.txt file.

For some functionality we need to use wrappers for Python code, such as reading in fits files or converting (x,y) -> (u,v). Thus, we need to use certain Python libraries. Thankfully, the setup for this is still pretty straightfoward. We use PyCall to run these snippets. If the Python snippets throw an error, run the following in the Julia REPL for each Python library:

```julia
using PyCall
pyimport("treecorr")
```
If you have a Conda Enviornment setup, you may find it easier to run 
```julia
using PyCall
pyimport_conda("treecorr", "tc") #tc is my choice of name and treecorr is what I am importing from my conda Enviornment 
```

On the off chance that none of these works, a final method may look like 
```julia
using PyCall
run(`$(PyCall.python) -m pip install --upgrade cython`)
run(`$(PyCall.python) -m pip install astropy`)
```

After the file contents are downloaded the user can run ```julia shopt.jl [configdir] [outdir] [catalog] [scifile]``` as stated above. Alternatively, they can run the shellscript that calls shopt in whatever program they are working with to create their catalog. For example, in a julia program you may use ```run(`./runshopt.sh [configdir] [outdir] [catalog] [scifile]`)```

## Program Architecture

shopt.jl 
> A runner script for all functions in this software

dataPreprocessing.jl
> A wrapper for python code to handle fits files and dedicated file to deal with data cleaning 

dataOutprocessing.jl
> Convert data into a summary.shopt file. Access this data with reader.jl. Produces some additional python plots.

reader.jl
> Get Point Spread Functions at an arbitrary (u,v) by reading in a summary.shopt file 

plot.jl 
> A dedicated file to handle all plotting

radialProfiles.jl 
> Contains analytic profiles such as a Gaussian Fit and a kolmogorov fit

analyticLBFGS.jl 
> Provides the necessary arguments (cost function and gradient) to the optimize function for analytic fits 

interpolate.jl 
> For Point Spread Functions that vary across the Field of View, interpolate.jl will fit a nth degree polynomial in u and v to show how each of the pixel grid parameters change across the (u,v) plane

outliers.jl 
> Contains functions for identifying and removing outliers from a list

powerSpectrum.jl
> Computes the power spectra for a circle of radius k, called iteratively to plot P(k) / k

kaisserSquires.jl
> Computes the Kaisser-Squires array to be plotted

runshopt.sh
> A shell script for running Shopt. Available so that users can run a terminal command in whatever program they are writing to run shopt. 

LICENSE
> MIT LICENSE

README.md
> User guide, Dependencies, etc.

index.md
> For official website

_config.yml
> Also for official website

imports.txt
> List of Julia Libraries used in the program

dependencies.jl 
> Download all of the imports from imports.txt automatically

## Known Issues
+ kolmogorov radial profile not yet functional

## Contributors
+ Edward Berman
+ Jacqueline McCleary

## Further Acknowledgements                                                                                                             
+ The Northeastern Cosmology Group for Their Continued Support and Guidance                                                           
+ The Northeastern Physics Department and Northeastern Undergraduate Research and Fellowships, for making this project possible with funding from the Northeastern Physics Co-Op Fellowship and PEAK Ascent Award respectively   
+ [David Rosen](https://github.com/david-m-rosen), who gave valuable input in the early stages of this project and during his course Math 7223, Riemannian Optimization
+ The COSMOS Web Collaboration for providing data from the James Webb Space Telescope and internal review

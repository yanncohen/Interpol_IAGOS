# Interpol-IAGOS
This package aims at assessing model simulations with IAGOS measurements.
The main task is a shell script (make_interpol_iagos.sh)
that treats the data, both simulation outputs and observations, to build two different products:
seasonal climatologies at every vertical grid level where cruise data are abundant,
and time series above a set of predefined regions, possibly in separated layers
as the upper troposphere or the lower stratosphere. One record dimension for time, and another dimension
for the season. The latter is five element-sized: the four seasons and the whole year.
All the routines are gathered in the path Pack_code/routines/grid_IAGOS_model.
#
How to use
#
Before launching, choose your parameters. 
For both tasks, the main program starts with all the options that the user can change, with their explanations.
For example, the simulation characteristics (the model name, the horizontal grid resolution,
the time resolution of the output), the variables to process or the start and end of the period to process.
The end of this 'user' zone is clearly specified. Everything else is automatic.
### Please read the comments given in this 'user' zone before running its program. ### 
They explain the program behaviour and the utility/consequences of each parameter you fix.

### Advanced specifications for bold users ###
If the user wants to increase the spectrum of variables or models that can be asked to the program,
then he has to add the corresponding information into the tables at this effect (in the folder grid_IAGOS_model):
table_variable_names_in_... and table_variable_units_in_...
There are 4 of them, two for observations and two for the simulation output. They contain information
on modelled/measured variables, necessary to read and convert automatically their fields from the data files.
- Concerning the simulation, the table_variable_name file contains the name of each variable, one column per model.
It also contains an output names column for homogenisation between the models, and a standard names column made to
match with the user request. The variable names in the latter are as clear as possible.
Concerning the observations, the table_variable_name file follows the same organisation, with one column per IAGOS package (IAGOS-MOZAIC, IAGOS-Core or IAGOS-CARIBIC) because the variable names differ from one package to another. Unless the database is updated, the user is not
concerned by this file.
- The table_variable_units file contains two columns per model/package: one for the units for each model/package,
one for the conversion factor to apply to homogenise the outputs. For variables which conversion does not consist in a product,
 the factor has to be set at 1 and the conversion has to be added manually in the code, as it is done for temperature in IAGOS files.

### Quick description

The script make_interpol_iagos.sh first calls the Fortran routine interpol_iagos.f90,
which interpolates the IAGOS data (in NetCDF files) onto the model grid,
and computes monthly means for the measured variables indicated in the user request.
It reads the surface pressure fields and the vertical coefficients to compute pressure levels, or simply reads the 3D pressure fields if they exist.
Then, it reads each IAGOS file and interpolates linearly each observation onto the grid (more exactly, following a reverse interpolation).
It also computes monthly means on each grid cell, using spatial weighting factors based on distances.
interpol_iagos.f90 generates monthly gridded data, both from IAGOS and from the model, with one file
per layer and per month.

make_interpol_iagos.sh then generates regional output files on the specified UT/LS
grid levels. For each region, there are:
- one IAGOS file, consisting in monthly-mean IAGOS data projected
onto the model grid. (IAGOS-DM)
- one model file, consisting in the results from the simulation after applying a mask, depending on the IAGOS sampling (this_model-M).
#
In make_interpol_iagos.sh, the header shows you the dates delimitating
the period you want to study. You also have to pick the model/simulation you
want to compare to the IAGOS data, and the location of the output
files. The header also shows several binaries. Each one of them
that is TRUE activates one part of the script, coded in the file content.

0/ First of all, the script calls a R routine (compute_resolution.R) that provides all the required grid characteristics.

1/ Call of interpol_iagos.f90:

The subroutines you need in the same repertory are:
- INT2CHAR.f90
- REAL2CHAR.f90
- compute_pressures.f90
- mean_values_interpol.f90
- standard_deviations_interpol.f90
- tonetcdf.F90

with the modules:
- global_var_interpol_mod.f90
- functions_mod.f90

The input repertories are:
- data/reference_values/this_model/, containing notably the two coefficient tables 
zvamh_nlev.txt and zvbmh_nlev.txt, to be able to generate the 3D pressure fields.
(nlev stands for the number of vertical grid levels)
- data/IAGOS_database/NetCDF/, where the IAGOS files (one file per flight)
are available until December 2017 for now.
Each repertory corresponds to a month, they are gathered in yearly repertories.
- data/models_output/this_model/this_configuration/this_experiment/this_time_resolution

2/ Preparing regional time series, ready for the analysis.

2.1/ Using a R routine (compute_grid_indexes.R) to derive the regions coordinates adapted to the grid resolution, both
in lon/lat degrees and in their corresponding indexes in the model grid.

2.2/ Using the latter as input for nco commands to extract the grid cells for each region.

2.3/ Call of regional_average_IAGOS.f90:

The subroutines you need in the same repertory are:
- INT2CHAR.f90
- REAL2CHAR.f90
- tonetcdf_time_series.F90

with the modules:
- global_var_interpol_mod.f90
- functions_mod.f90

3/ Preparing climatologies, ready for the analysis.

Using nco commands to concatenate throughout the years and to compute seasonal means,
and yearly means also. The first and last months defined by the user are readapted
in a way that allows an equivalent period sampling between the seasons.
#

Author: Yann Cohen

yann.cohen.09@gmail.com

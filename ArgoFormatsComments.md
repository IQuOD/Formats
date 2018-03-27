# The Argo Format

This is a review of the Argo file formats (as per version 3.2 of the [Argo User's Manual](http://dx.doi.org/10.13155/29825)) and its potential to be adapted for IQuOD data products.

## The core-Argo single cycle format
This format is for files containing _one or more_ profiles from a single cycle of a single Argo float. They contain only the standard CTD parameters (pressure, temperature, salinity, conductivity). Additional parameters measured during the same cycle are stored in separate "B-Argo" files.

### Structure
All parameters included in a file are measured on the "same vertical sampling scheme and at the same location and time" (Argo manual, p19). This implies that all profiles in the file have the same number of vertical levels, though the exact pressure (as measured) at a given level may differ from one profile to the next. The basic structure of the main CTD data and coordinate variables in the file is (dimension sizes are just examples):

```
dimensions:
    N_PROF = 2 ;                    // number of profiles
    N_LEVELS = 229 ;                // number of levels in each profile

variables:
    double JULD(N_PROF) ;           // axis = "T"
    double LATITUDE(N_PROF) ;       // axis = "Y"
    double LONGITUDE(N_PROF) ;      // axis = "X"
    float PRES(N_PROF, N_LEVELS) ;  // axis = "Z"
    float TEMP(N_PROF, N_LEVELS) ;
    float PSAL(N_PROF, N_LEVELS) ;
```

For each of the CTD parameters (`PRES`, `TEMP`, `PSAL`, `CNDC`), QC flags and calibrated values are stored in corresponding variables with the same dimensions, e.g.

```
    char TEMP_QC(N_PROF, N_LEVELS) ;
    float TEMP_ADJUSTED(N_PROF, N_LEVELS) ;
    char TEMP_ADJUSTED_QC(N_PROF, N_LEVELS) ;
    float TEMP_ADJUSTED_ERROR(N_PROF, N_LEVELS) ;
```

Additinoal information about each profile is provided in variables with the `N_PROF` dimension. Other metadata relating to calibrations applied to individual parameters requires additional dimentions, and all the string-valued variables require extra dimensions to define their lengths:
```
    N_CALIB = 1 ;
    STRING2 = 2 ;
    STRING4 = 4 ;
    STRING8 = 8 ;
    STRING16 = 16 ;
    STRING32 = 32 ;
    STRING64 = 64 ;
    STRING256 = 256 ;
    DATE_TIME = 14 ;
    N_PARAM = 3 ;
    N_HISTORY = UNLIMITED ; // (6 currently)
```


### Comments (Marty)
+ The raw value of a measured parameter, e.g. temperature, is stored in the variable with the simplest name `TEMP`, while the calibrated values are in `TEMP_ADJUSTED`. Worse, when no adjusted values are available (for real-time data), the variable `TEMP_ADJUSTED` is still present, but contains only FillValues.
+ All the metadata described as "general info on the profile file" is stored in variables. Why not make these global attributes?
+ Date/time-valued variables are represented as strings, rather than the [CF standard](http://cfconventions.org/Data/cf-conventions/cf-conventions-1.7/cf-conventions.html#time-coordinate) way of representing time using floating-point values.
+ All calibration info, including numerical parameters, are stored as string variables.
+ Detailed history also recorded in string variables.


## B-Argo profile format
Similar to the core-Argo profile format, but includes _all_ parameters measured by a float _except_ temperature, salinity and conductivity. The raw pressure variable (`PRES`) from the core-Argo file is included as a link between the parameters in the two files. The dimensions `N_PROF` and `N_LEVELS` are also the same in the two files. Any parameter values that were not recorded in a particular profile are filled with FillValues.


## Example
The two txt files contain the headers for an Argo float that has bio parameters. The first 'vanilla' file is [argo_nc.txt](argo_nc.txt) and contains the P/T/S data for the float. The second 'B' file [argo_bio_nc.txt](argo_bio_nc.txt) shows the header for the float with the additional bio parameters.


## Other Argo formats
Other file formats used by Argo, such as the trajectory, metadata, and technical files, are specific to the floats and currently of no particular interest for IQuOD.




# Adapting Argo format for IQuOD

The Argo format is set up for single profile files. There are metadata, technical and trajectory files that are also produced per float. For the IQuOD project, we would use the profile file structure as the basis of the format, not the other file types.

If a profile has parameters on the same pressure axis (eg, temperature, salinity), this would be a straightforward single profile file.

Argo has biogeochemical floats with profiles data collected on different pressure axes. Argo handles this by producing two files per profile. One is a 'B' file and contains the bio float data on its own axis. The other is the regular file with the temperature and salinity data on its pressure axis.

We need to consider that there are a number of profiles in the global historical database that have more than one parameter on different pressure axes. Providing a 'B' file for these profiles would make sense.


The Argo format is set up for single profile files. There are metadata, technical and trajectory files that are also produced per float. For the IQuOD project, we would use the profile file structure as the basis of the format, not the other file types.

If a profile has parameters on the same pressure axis (eg, temperature, salinity), this would be a straightforward single profile file.

Argo has biogeochemical floats with profiles data collected on different pressure axes. Argo handles this by producing two files per profile. One is a 'B' file and contains the bio float data on it's own axis. The other is the regular file with the temperature and salinity data on it's pressure axis.

We need to consider that there are a number of profiles in the global historical database that have more than one parameter on different pressure axes. Providing a 'B' file for these profiles would make sense.

The two txt files contain the headers for an Argo float that has bio parameters. The first 'vanilla' file is 'argo_nc.txt' and contains the P/T/S data for the float. The second 'B' file 'argo_bio_nc.txt' shows the header for the float with the additional bio parameters.

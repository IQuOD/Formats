This document will describe the requirements for IQuOD file formats, and make recommendations for how these requirements can be met by adapting the Argo format.


# Requirements

The IQuOD data products will be delivered in a format based on the Argo NetCDF format. IQuOD quality control and uncertainty estimates will primarily be applied to pressure, temperature and salinity, but other measured parameters in the source data will be included in the output products.

For the forseeable future IQuOD will only deal with _profile_ data. Other "shapes", such as time-series do not need to be accommodated in this format.

The main requirements are:
* Include all parameters measured in the source data;
* Include all key metadata from the source data (QC flags, etc...);
* Include IQuOD-speecific metadata (flags, uncertainties, I-metadata);
* Compatible with Argo data so that existing software can read the format with minimal modifications.

It may be helpful to define two variants for IQuOD:

1. A simple format containing a single profile on a single pressure axis
  - Same as core-Argo, with `N_PROF = 1`, and additional IQuOD metadata
  - Include all variables on the same pressure axis (not just pressure, temperature & salinity)

2. An aggregated, multi-profile format
   - A combination of the Argo "core" and "B" formats, with additional metadata
   - Allow many profiles from different sources, with different pressure levels, different measured parameters and different (source specific) metadata fields.


# Questions

### Is a single-profile format actually useful?
This would be the simplest to write and read, but would really hold just one single profile on a single set of pressure levels. This is *less than some core-Argo files* (which can hold multiple profiles from a single cycle)!

### TEMP vs TEMP_ADJUSTED
Would Argo data users find it hard to adopt the IQuOD format if we redefine the variable **TEMP** to mean the calibrated or best availabe value of temperature, instead of the raw value?

### Is there a need for a separate "B" format?

In Argo, the B format is used for separate files that contain all parameters other than temperature, conductivity, and salinity. The IQuOD requirement is to carry through all these additional variables, which we could do by replicating the Argo B format. The only content of these files that will be affected by IQuOD procedures (at least in the near future) is the pressure variable and the intelligent metadata. (Is that right?)

The question is, for the IQuOD products, does it make sense to create a whole separate collection of these B files just to contain data that is not the primary product of IQuOD?

The alternative would be to include _all_ parameters ("core" and "B") in the same multi-profile files.


### How to stucture aggregated files?
The "natural" way to store measured values from multiple profiles is in a two-dimensional array, with a profile index, and a pressure/level index, as is done in the [core-Argo format](https://github.com/IQuOD/Formats/blob/master/ArgoFormatsComments.md#structure). In the general case where profiles have different numbers of levels, the CF conventions call this [incomplete multidimensional array representation](http://cfconventions.org/cf-conventions/v1.6.0/cf-conventions.html#_incomplete_multidimensional_array_representation). It is simple to read data from such files, but they can be large and contain lots of fill values (because their size is set by the profile with the maximum number of levels). The extra space taken up by the fill values is largely eliminated when the file is compressed, though this comes at the cost slightly increased access time.

The alternative is the [contiguous ragged array representation](http://cfconventions.org/cf-conventions/v1.6.0/cf-conventions.html#_contiguous_ragged_array_representation), which packs all the profiles "end-to-end" into a single dimension, removing the need for fill values. Reading values from this structure is slightly more complex because of the need to keep track of the length of each profile.

### Which NetCDF version and options to use?
Should IQuOD products be stored in NetCDF3 or NetCDF4 files? If the latter, should we use optional features such as chunking and compression to reduce the file sizes? These features would be most useful for mutli-profile files with a simple multi-dimensional structure, leading to many fill values. They are less relevant for ragged-array files.

For a single-profile format, we may as well use NetCDF3 for maximum complatibility.


# Recommendations

## Data variables

* The final, "best estimate" values for each measured parameter (e.g. temperature) should be stored in the variable with the simplest name (e.g. **TEMP**, instead of **TEMP_ADJUSTED**). These values should always be defined, even if equal to the raw value (i.e. no adjustment was made). They should only contain a FillValue where the original measurement is missing or bad.

* The original, raw values of the parameter should be stored in **<PARAM>_RAW** (e.g. **TEMP_RAW**).


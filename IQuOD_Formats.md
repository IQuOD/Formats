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
Please discuss these on the [issues](https://github.com/IQuOD/Formats/issues) pages.
* [Is a single-profile format actually useful?](https://github.com/IQuOD/Formats/issues/1)
* [TEMP vs TEMP_ADJUSTED](https://github.com/IQuOD/Formats/issues/2)
* [Is there a need for a separate "B" format?](https://github.com/IQuOD/Formats/issues/3)
* [How to stucture aggregated files?](https://github.com/IQuOD/Formats/issues/4)
* [Which NetCDF version and options to use?](https://github.com/IQuOD/Formats/issues/5)


# Recommendations

## Data variables

* The final, "best estimate" values for each measured parameter (e.g. temperature) should be stored in the variable with the simplest name (e.g. **TEMP**, instead of **TEMP_ADJUSTED**). These values should always be defined, even if equal to the raw value (i.e. no adjustment was made). They should only contain a FillValue where the original measurement is missing or bad.

* The original, raw values of the parameter should be stored in **<PARAM>_RAW** (e.g. **TEMP_RAW**).


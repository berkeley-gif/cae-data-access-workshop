# Downloading AE data using the AWS Command Line Interface (CLI)

The free [AWS Command Line Interface (CLI)](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) tool is useful for bulk downloading entire directory structures of data at once, or downloading Zarr stores (which are directory structures) to a local machine. The AWS CLI is an open source tool enabling the use of the command-line shell to access and interact with the [Analytics Engine S3 bucket](https://cadcat.s3.amazonaws.com/index.html) directly. This tool simplifies downloading all the data in a dataset. 

Best Use: 

Downloading large amounts of data, including multiple models, simulations, and/or resolutions. Can be configured to filter for specific models, variables, resolutions, etc., based on the storage “directory” structure of the data in S3.

Limitations: 

Must download the entire dataset, whether it is NetCDF or Zarr store. Can not spatially or temporally filter the data using AWS CLI, thus resulting in the possibility of downloading more data than is necessary. This solution does not require an Analytics Engine login but does require some familiarity with shell scripting.

Example Usage: 

Natively, AWS S3 does not store data in a traditional directory structure but instead uses keys to the binary data stored there. The AWS Explorer represents the data keys as directories for convenience. Users can first utilize either the  [AWS Explorer](https://cadcat.s3.amazonaws.com/index.html) or [Data Catalog](https://analytics.cal-adapt.org/data/catalog/) to find the path to the data of interest. The following is an example of listing the bucket data using AWS CLI to display the variables available for this model:

Download WRF t2 monthly using AWS CLI command:
```
# List variables for WRF monthly for CESM2 SSP3-7.0
aws s3 ls --no-sign-request s3://cadcat/wrf/ucla/cesm2/ssp370/mon/
# Download Air temperature at 2m
aws s3 cp s3://cadcat/wrf/ucla/cesm2/ssp370/mon/t2/d01/ wrf/ucla/cesm2/ssp370/mon/t2/d01/--no-sign-request --recursive
```

```
import xarray as xr
ds = xr.open_zarr('wrf/ucla/cesm2/ssp370/mon/t2/d01')
import rioxarray as rio
import pyproj
ll_to_lambert = pyproj.Transformer.from_crs(crs_from="epsg:4326",crs_to=result.rio.crs,always_xy=True)
x, y = ll_to_lambert.transform(-121.23,38.33)
ds['TX99p'].sel(x=x,y=y,method='nearest').values
x, y = ll_to_lambert.transform(-122.27,37.87))
ds['TX99p'].sel(x=x,y=y,method='nearest').values
x, y = ll_to_lambert.transform(-118.24,34.05)
ds['TX99p'].sel(x=x,y=y,method='nearest').values
```

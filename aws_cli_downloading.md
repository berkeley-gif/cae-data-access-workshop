# Downloading AE data using the AWS Command Line Interface (CLI)

The free [AWS Command Line Interface (CLI)](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) tool is useful for bulk downloading entire directory structures of data at once, or downloading Zarr stores (which are directory structures) to a local machine. The AWS CLI is an open source tool enabling the use of the command-line shell to access and interact with the [Analytics Engine S3 bucket](https://cadcat.s3.amazonaws.com/index.html) directly. This tool simplifies downloading all the data in a dataset. 

Best Use: 

Downloading large amounts of data, including multiple models, simulations, and/or resolutions. Can be configured to filter for specific models, variables, resolutions, etc., based on the storage “directory” structure of the data in S3.

Limitations: 

Must download the entire dataset, whether it is NetCDF or Zarr store. Can not spatially or temporally filter the data using AWS CLI, thus resulting in the possibility of downloading more data than is necessary. This solution does not require an Analytics Engine login but does require some familiarity with shell scripting.

Example Usage: 

Natively, AWS S3 does not store data in a traditional directory structure but instead uses keys to the binary data stored there. The AWS Explorer represents the data keys as directories for convenience. Users can first utilize either the  [AWS Explorer](https://cadcat.s3.amazonaws.com/index.html) or [Data Catalog](https://analytics.cal-adapt.org/data/catalog/) to find the path to the data of interest.

You can use the CLI `s3 ls` command to list the variables available for WRF, monthly, for this model and SSP.
```
:: List variables for WRF monthly for CESM2 SSP3-7.0 using the `s3 ls` command:
aws s3 ls --no-sign-request s3://cadcat/wrf/ucla/cesm2/ssp370/mon/
```
The `--no-sign-request` option tells the CLI no authentication is required, since this is a public S3 bucket.

You can then use the CLI to find out the size of a dataset before downloading it. The `summarize` option calculates the total size of the files.

```
:: Calculate size of data on S3
aws s3 ls --summarize --human-readable --recursive --no-sign-request s3://cadcat/wrf/ucla/miroc6/ssp370/1hr/t2/d03
:: This will return 207GB
```

For this hourly data it is 207 gigabytes, which will take quite some time to download.

Now you can download the WRF monthly data using the `s3 cp` command.
```
:: Download WRF Air temperature at 2m
aws s3 cp s3://cadcat/wrf/ucla/cesm2/ssp370/mon/t2/d01/ wrf/ucla/cesm2/ssp370/mon/t2/d01/ --no-sign-request --recursive
```
This is quick because it is monthly data.
And here is an example LOCA2 download.
```
:: Download LOCA2 Maximum air temperature at 2m
aws s3 cp s3://cadcat/loca2/ucsd/fgoals-g3/ssp585/r1i1p1f1/mon/tasmax/d03/ loca2/ucsd/fgoals-g3/ssp585/r1i1p1f1/mon/tasmax/d03/ --no-sign-request --recursive
```
Another example which I won’t run uses the `--include` and `--exclude` options to download netCDF temperature data for all LOCA2 models.

```
aws s3 cp s3://cadcat/loca2/aaa-ca-hybrid . --no-sign-request --recursive --exclude '*' --include '*tasmax*'
:: This will return 34GB of data
```
This command will get 34GB of data.

Let’s now take a look at this data in Python.

First we import xarray, a multi dimensional array processing package. It can open netCDF as well as Zarr.
```
import xarray as xr
```

We then use the `open_zarr()` command.

```
ds = xr.open_zarr('wrf/ucla/cesm2/ssp370/mon/t2/d01')
ds
```

The dataset is now a Python object and you can see the coordinates: x, y and time. Looking at the x and y you can see that they are large numbers. That is because WRF is stored in a custom Lambert Conformal Conic projection, whereas the LOCA2 data are stored in lat and long. In order to use lat and long we need to transform the coordinate into the correct projection.

We can use these packages and code to do that.
```
import rioxarray as rio
import pyproj
```

```
# Function to transform the coordinates
ll_to_lambert = pyproj.Transformer.from_crs(crs_from="epsg:4326", crs_to=ds.rio.crs, always_xy=True)
```

The Lambert projection info is stored as an attribute of the dataset.

Now we can look at the values for the dataset at Sacramento’s lat and long.
```
# Show values at Sacramento
x, y = ll_to_lambert.transform(-121.23, 38.33)
ds['t2'].sel(x=x, y=y, method='nearest').values
```
The temperature values are in Kelvin for each of the time steps.

We can do the same for Los Angeles and see that the values are different.
```
# Show values at Los Angeles
x, y = ll_to_lambert.transform(-118.24, 34.05)
ds['t2'].sel(x=x, y=y, method='nearest').values
```
Instead of downloading the data we can actually load it directly from S3, using the same command but instead using the S3 path to the data.
```
ds = xr.open_zarr(
's3://cadcat/wrf/ucla/cesm2/ssp370/mon/t2/d01/', storage_options={'anon': True}
)
ds
```
We can then run the same commands and get the same answers but without actually downloading the whole dataset.

Let's get Los Angeles data from S3 directly.

```
# Show values at Los Angeles
x, y = ll_to_lambert.transform(-118.24, 34.05)
ds['t2'].sel(x=x, y=y, method='nearest').values
```
This covers the basics of using the AWS CLI to download Analytics Engine data from S3.

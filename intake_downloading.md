# Access Cal-Adapt: Analytics Engine Zarr data using Intake Python package

As part of building [climakitae](https://github.com/cal-adapt/climakitae), we created an [intake](https://github.com/intake/intake) data catalog, that allows for easy programmatic access to the data used on the Cal-Adapt: Analytics Engine, and is specifically an implementation of [intake-ESM](https://intake-esm.readthedocs.io/en/stable/index.html) (Earth System Model), which is designed for climate data. This catalog can be opened using Python by knowing the URL of the catalog file. 

Best Use:

Accessing the **cae-collection** of data in Python without having to download the dataset completely. Allows for spatial and temporal filtering of the data before exporting to the local filesystem. It can also be used to analyze large datasets with Xarray and Dask, and to convert the data to NetCDF if there is sufficient memory.

Limitations:

Can only help access data that is in the **cae-collection** intake catalog. Other datasets in the **cadcat** S3 bucket have to be referenced manually using this option. To access all the data in the bucket, look into the **climakitae** python package.

Example Usage: 

```
import intake
```
Import the intake package

```
cat = intake.open_esm_datastore(
    'https://cadcat.s3.amazonaws.com/cae-collection.json'
)
```

Open the intake ESM data catalog stored in the S3 bucket

```
cat2 = cat.search(activity_id='LOCA2', table_id='mon')
cat2.unique()
```

You can filter for LOCA2 monthly and see all the options. You can then see the list of variables by running this:

```
cat2.unique()['variable_id']
```

From this list let us choose the precipitation variable, CESM2-LENS model, and SSP3-7.0 simulation.

```
cat2 = cat.search(activity_id='LOCA2',
                  table_id='mon',
                  variable_id='pr',
                  source_id='CESM2-LENS',
                  experiment_id='ssp370')
cat2.unique()
cat2
```

LOCA2 has an extra layer over WRF - member_id which are different runs of the model. To get to one dataset we can choose a member_id.

```
cat2 = cat.search(activity_id='LOCA2',
                  table_id='mon',
                  variable_id='pr',
                  source_id='CESM2-LENS',
                  experiment_id='ssp370',
                  member_id='r1i1p1f1')
cat2.unique()
cat2
```

We can now retrieve the dataset from S3.

```
dset_dict = cat2.to_dataset_dict(zarr_kwargs={'consolidated':True}, storage_options={'anon':True})
dset_dict
```

This function can return multiple datasets and is returned as a dictionary. We can get the Dataset object by:

```
ds = dset_dict['LOCA2.UCSD.CESM2-LENS.ssp370.mon.d03']
```

Now that we have the precipation data let us calculate yearly totals in inches. Natively, the pr variable is a rate with kg m-2 s-1 as units so we need to convert to inches:

```
year = ds['time'].dt.year
ds = ds.assign_coords({'year':year})
```

This adds year as a coordinate to the data

```
pr = ds['pr']
pr = pr * pr['time'].dt.days_in_month
pr = (pr * 86400) / 25.4
```

Converts to a xarray DataArray and then converts the units. We can then summarize the data by year.

```
result = pr.groupby('year').sum('time')
result.sel(lat=38.33,lon=-121.23,method='nearest').values
```

You can see the wetter/dryer year variance here. Let us get the 30yr average monthly rainfall in inches for the beginning of the timeseries and the end:

```
time1 = pr.sel(time=slice('2015-01-01','2044-12-31'))
time2 = pr.sel(time=slice('2070-01-01','2099-12-31'))
avg1 = time1.mean('time')
avg2 = time2.mean('time')
avg1.sel(lat=38.33,lon=-121.23,method='nearest').values
avg2.sel(lat=38.33,lon=-121.23,method='nearest').values
```

You can see that the rainfall average increases at end of century.

Finally, you can export the yearly averaged data:

```
ds.to_netcdf('loca2.ucsd.cesm2-lens.ssp370.mon.d03.nc', encoding={k: {'zlib': True, 'complevel': 6} for k in ds})
```

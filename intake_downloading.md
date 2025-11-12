# Access Cal-Adapt: Analytics Engine Zarr data using Intake Python package

As part of building [climakitae](https://github.com/cal-adapt/climakitae), we created an [intake](https://github.com/intake/intake) data catalog, that allows for easy programmatic access to the data used on the Cal-Adapt: Analytics Engine, and is specifically an implementation of [intake-ESM](https://intake-esm.readthedocs.io/en/stable/index.html) (Earth System Model), which is designed for climate data. This catalog can be opened using Python by knowing the URL of the catalog file. 

Best Use:

Accessing the **cae-collection** of data in Python without having to download the dataset completely. Allows for spatial and temporal filtering of the data before exporting to the local filesystem. It can also be used to analyze large datasets with Xarray and Dask, and to convert the data to NetCDF if there is sufficient memory.

Limitations:

Can only help access data that is in the **cae-collection** intake catalog. Other datasets in the **cadcat** S3 bucket have to be referenced manually using this option. To access all the data in the bucket, look into the **climakitae** python package.

Example Usage: 

To get started we simply import intake into Python
```
import intake
```

Then we can connect to the Json file in S3 using the `open_esm_datastore()` command.
```
cat = intake.open_esm_datastore('https://cadcat.s3.amazonaws.com/cae-collection.json')
```
As you can see at the top are LOCA2 and WRF.
```
cat.unique()
```

We can then search the catalog using the search() function. Let us search for LOCA2 monthly data and then show the unique entries of that query.
```
cat2 = cat.search(activity_id='LOCA2', table_id='mon')
cat2.unique()
```
`activity_id` is LOCA2.

`institution_id` is UCSD but it can be multiple.

`source_id` are models.

`experiment_id` are scenarios.

`member_id` are the runs - WRF will have only one.

`table_id` are the time scales - yrmax, monthly, daily, or hourly, but we already filtered for monthly.

`variable_id` are the variables.

`grid_label` are the spatial resolutions, for LOCA2 all the data are at 3 km.

`paths` are the S3 URLs to the data.

You can use this command to list the available variables.

```
cat2.unique()['variable_id']
```

Let us pick PR, which is precipitation. Let's also filter for this model and scenario.

```
cat2 = cat.search(activity_id='LOCA2',
                  table_id='mon',
                  variable_id='pr',
                  source_id='CESM2-LENS',
                  experiment_id='ssp370')
cat2.unique()
cat2
```

If we now look at what we have filtered you can see it has become far more refined. We still need to deal with the fact that LOCA2 has multiple runs.down to a single dataset.

We can just select one and we have filtered to a single dataset.
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

We can now retrieve a connection to that data set. And to do that, we run this `to_dataset_dict()` command.

```
dset_dict = cat2.to_dataset_dict(zarr_kwargs={'consolidated':True}, storage_options={'anon':True})
dset_dict
```

We now have a dataset dictionary. When data is retrieved using intake, it is returned as a dictionary. That way you can have multiple data sets returned as a collection.

To get the one data set that we have we can simply pull out that object in the dictionary.

```
ds = dset_dict['LOCA2.UCSD.CESM2-LENS.ssp370.mon.d03']
ds
```
We now have the dataset as an object.

Let's now do something with this data set. Let's calculate the yearly totals in inches.

Natively the precipitation variable is actually a rate, its units are kilograms per square meter per second. The first thing we need to do is attach the year as a coordinate to the data set. We can do that with this command.

```
year = ds['time'].dt.year
ds = ds.assign_coords({'year':year})
ds
```
So now if we look we have the year attached. This is so we can group by year.

We can now do the unit conversion to inches per year. This does not take any time, because we are just staging the calculation.

```
ds['pr'] = ds['pr'] * ds['time'].dt.days_in_month
ds['pr'] = (ds['pr'] * 86400) / 25.4
```
When we do this next command the actual calculation occurs.

Let’s ask what the values are for Sacramento.

```
pr_by_year = ds['pr'].groupby('year').sum('time')
pr_by_year.sel(lat=38.33, lon=-121.23, method='nearest').values
```
You can see there are values for each year in inches. This is California so there are dry and wet years.

Maybe we are then curious what the rainfall trend is? One simplistic way to calculate this would be to do a 30 year average at the beginning of the time series and a 30 year average at the end of the time series. These commands do that. We slice the time axis and then take the average of that time slice.

```
avg1 = ds['pr'].sel(time=slice('2015-01-01', '2044-12-31')).mean('time')
avg2 = ds['pr'].sel(time=slice('2070-01-01', '2099-12-31')).mean('time')
```

We can then ask for the values at Sacramento.

```
avg1.sel(lat=38.33, lon=-121.23, method='nearest').values
```
```
avg2.sel(lat=38.33, lon=-121.23, method='nearest').values
```

And we can see the monthly average is increasing for this particular model and scenario.

Finally, you might want to export the monthly averaged data. To do that you can run the xarray to_netcdf() command to export the data. You give it a file name, and we can add compression to the data. It will take a minute to run and the file is about 400 megabytes.

```
ds.to_netcdf('loca2.ucsd.cesm2-lens.ssp370.mon.d03.nc', encoding={k: {'zlib': True, 'complevel': 6} for k in ds})
```

That is an introduction to using intake to access the Analytics Engine data.

# Accessing or downloading data using the  climakitae Python package

A powerful Python toolkit for climate data retrieval and analysis has been developed as part of the AE project, [climakitae](https://github.com/cal-adapt/climakitae). The main features of the toolset are:

- Comprehensive Climate Data Access: Retrieve climate variables from hosted climate models
- Downscaled Climate Models: Access dynamical (WRF) and statistical (LOCA2) downscaling methods
- Spatial Analysis Tools: Built-in support for geographic subsetting and spatial aggregation
- Climate Indices: Calculate heat indices, warming levels, and extreme event metrics
- Flexible Data Export: Export to NetCDF, CSV, and Zarr
- GUI Integration: Works seamlessly with [climakitaegui](https://github.com/cal-adapt/climakitaegui) for interactive analysis

Climakitae includes built-in methods to aid in the retrieval of data from the cadcat S3 bucket that are more intuitive and user-friendly than the previous Python examples. 

Best Use:

Powerful functionality allows users to retrieve data by warming level, by spatial and/or temporal filtering, by spatial and/or temporal aggregation, by derived variable, or by concatenated timeseries for historical and future scenarios. Allows users to utilize all of the CA: AE software functionality contained within the CA: AE JupyterHUB, especially when combined with the notebooks developed for the HUB: [cae-notebooks](https://github.com/cal-adapt/cae-notebooks) 

Limitations: 

Limited to the resources on the local computer (RAM, CPU, network), dataset processed from the original (simulations and time series concatenated, metadata added, warming levels calculated.

Example Usage: 

To start with, we need to import climakitae and climakitaegui.

```
import climakitae as ck
import climakitaegui as ckg
```

Then we want to import the `get_data()` and `get_data_options()` functions. `get_data()` is the main function that will retrieve the data from the S3 bucket. `get_data_options()` is a helper function that will help us to discover what data is in the data catalog.

```
from climakitae.core.data_interface import get_data
from climakitae.core.data_interface import get_data_options
```

We can run the `get_data_options()` function, and it will return a Pandas Dataframe.

```
get_data_options()
```

This is all sitting on top of intake and should look familiar except it is more presentable. And the more obscure naming nomenclature used in intake has been replaced with a more user friendly language, such as `downscaling_method` for `activity_id`. Statistical means LOCA2, and Dynamical means WRF. We are at the top of the catalog so there are 1200 rows of data in this dataframe.

Let’s refine what we are looking for by choosing LOCA2 daily Maximum air temperature at 2m which is the tasmax variable.

```
get_data_options(downscaling_method="Statistical", timescale='daily', variable="Maximum air temperature at 2m")
```

For this query there are several scenarios available and the data are at 3km resolution.

There is also a helper function for finding spatial subsetting options - `get_subsetting_options()`. We can use that to see that we have all 58 California Counties and you can reference them by name.
```
from climakitae.core.data_interface import get_subsetting_options
get_subsetting_options(area_subset="CA counties")
```

These predefined geometries can be used to spatially filter the data, and the software takes care of the WRF projection issue for you. There are several different geometries for filtering, such as states, counties, watersheds and others.

Now let’s get some data. Let's go for LOCA2, daily, maximum air temperature at three kilometers resolution for SSP 3-7.0, for Sacramento County only, and for the time slice of 2070 to 2100. And let us return the data in degrees centigrade instead of Kelvin.
```
data = get_data(downscaling_method="Statistical",
                timescale="daily",
                variable="Maximum air temperature at 2m",
                resolution="3 km",
                scenario="SSP 3-7.0",
                cached_area="Sacramento County",
                time_slice=(2070,2100),
                units="degC")
data
```

We can see here that we have maximum air temperature at two meters. For one scenario and 62 simulations. It has brought back all the simulations for this data.

Let’s select for one particular simulation, MIROC6.

```
data1 = data.sel(simulation='LOCA2_MIROC6_r1i1p1f1')
data1
```

As an example, let’s calculate the annual averages for the data. Again we will assign year to the data as an additional coordinate.

```
year = data1['time'].dt.year
data1 = data1.assign_coords({'year':year})
data1
```

And then do the average and ask it for the values at Sacramento.

```
data2 = data1.groupby('year').mean('time')
data2
data2.sel(lat=38.33,lon=-121.23,method='nearest').values
```

We can see here that we have the temperature in centigrade for each year from 2070 to 2100, for the city of Sacramento.

We can now use this command to load the entire dataset we created into memory so we can visualize it.

```
data2 = ck.load(data2)
```

Now that we have the data loaded into memory, we can use the climakitaegui function called `view()`. This creates a simple visualization of the data.

```
ckg.view(data2)
```

We can see here this is Sacramento County. And if you hover over the map, you get values of individual cells. And you can scroll through the time series to see how the data changes.

We can export the data using climakitae. It can export to netCDF, CSV, and Zarr.

That's the basics of how to download data from the S3 bucket using climatekitae.

# Accessing or downloading data using the  climakitae Python package

A powerful Python toolkit for climate data retrieval and analysis has been developed as part of the AE project, climakitae. The main features of the toolset are:

- Comprehensive Climate Data Access: Retrieve climate variables from hosted climate models
- Downscaled Climate Models: Access dynamical (WRF) and statistical (LOCA2) downscaling methods
- Spatial Analysis Tools: Built-in support for geographic subsetting and spatial aggregation
- Climate Indices: Calculate heat indices, warming levels, and extreme event metrics
- Flexible Data Export: Export to NetCDF, CSV, and Zarr
- GUI Integration: Works seamlessly with climakitaegui for interactive analysis

Climakitae includes built-in methods to aid in the retrieval of data from the cadcat S3 bucket that are more intuitive and user-friendly than the previous Python examples. 

Best Use:

Powerful functionality allows users to retrieve data by warming level, by spatial and/or temporal filtering, by spatial and/or temporal aggregation, by derived variable, or by concatenated timeseries for historical and future scenarios. Allows users to utilize all of the CA: AE software functionality contained within the CA: AE JupyterHUB, especially when combined with the notebooks developed for the HUB: cae-notebooks 

Limitations: 

Limited to the resources on the local computer (RAM, CPU, network), dataset processed from the original (simulations and time series concatenated, metadata added, warming levels calculated.

Example Usage: 

First import climakitae and climakitaegui (a helper package for gui and visualization)

```
import climakitae as ck
import climakitaegui as ckg
```

Next let us explicitly import the get_data and get_data_options functions

```
from climakitae.core.data_interface import get_data
from climakitae.core.data_interface import get_data_options
```

Running get_data_options() returns a Pandas dataframe that is similar to intake interface but with less obscure nomenclature

```
get_data_options()
```

We can refine what we are looking for by specifying some parameters:

```
get_data_options(downscaling_method="Statistical", timescale='daily', variable="Maximum air temperature at 2m")
```

This is LOCA2-Hybrid daily air temperature. You can see there are several simulations to choose from. Now let us look at spatial filtering options:

```
from climakitae.core.data_interface import get_subsetting_options
get_subsetting_options(area_subset="CA counties")
```

You can see the list of 58 California counties that can be used to clip the data. Let us grab SSP 3-7.0 for Sacramento County:

```
data = get_data(downscaling_method="Statistical",
                timescale="daily",
                variable="Maximum air temperature at 2m",
                resolution="3 km",
                scenario="SSP 3-7.0",
                cached_area="Sacramento County")
data
```

You can see from the time range and simulations that we got both the historical and future time range and all 62 simulations. Let us focus on 1 simulation, MIROC6:

```
data1 = data.sel(simulation='LOCA2_MIROC6_r1i1p1f1')
data1
```

Now let us calculate the annual averages for this data:

```
year = data1['time'].dt.year
data1 = data1.assign_coords({'year':year})
data1
```

Add year as a coordinate.

```
data2 = data1.groupby('year').mean('time')
data2
data2.sel(lat=38.33,lon=-121.23,method='nearest').values
```

We can now load the data into memory

```
ck.load(data2)
```

And then make a map of the data:

```
ckg.view(data2)
```

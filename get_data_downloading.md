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

import climakitae as ck
import climakitaegui as ckg
from climakitae.core.data_interface import get_data
from climakitae.core.data_interface import get_data_options
get_data_options()
get_data_options(downscaling_method="Statistical", timescale='daily', variable="Maximum air temperature at 2m")
from climakitae.core.data_interface import get_subsetting_options
get_subsetting_options(area_subset="CA counties")
data = get_data(downscaling_method="Statistical",
                timescale="daily",
                variable="Maximum air temperature at 2m",
                resolution="3 km",
                scenario="SSP 3-7.0",
                cached_area="Sacramento County")
data
data1 = data.sel(simulation='LOCA2_MIROC6_r1i1p1f1')
data1
year = data1['time'].dt.year
data1 = data1.assign_coords({'year':year})
data1
data2 = data1.groupby('year').mean('time')
data2
data2.sel(lat=38.33,lon=-121.23,method='nearest').values
ck.load(data2)
ckg.view(data2)

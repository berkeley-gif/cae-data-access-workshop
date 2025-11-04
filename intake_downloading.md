import intake
cat = intake.open_esm_datastore(
    'https://cadcat.s3.amazonaws.com/cae-collection.json'
)
cat2 = cat.search(activity_id='LOCA2', table_id='mon')
cat2
cat2 = cat.search(activity_id = 'LOCA2',table_id='mon',variable_id='pr',source_id='CESM2-LENS',experiment_id='ssp370',member_id='r1i1p1f1')
dset_dict = cat2.to_dataset_dict(zarr_kwargs={'consolidated':True}, storage_options={'anon':True})
ds = dset_dict['LOCA2.UCSD.CESM2-LENS.ssp370.mon.d03']
year = ds['time'].dt.year
ds = ds.assign_coords({'year':year})
pr = ds['pr']
pr = pr * pr['time'].dt.days_in_month
pr = (pr * 86400) / 25.4
result = pr.groupby('year').sum('time')
result.sel(lat=38.33,lon=-121.23,method='nearest').values
time1 = pr.sel(time=slice('2015-01-01','2044-12-31'))
time2 = pr.sel(time=slice('2070-01-01','2099-12-31'))
avg1 = time1.mean('time')
avg2 = time2.mean('time')
avg1.sel(lat=38.33,lon=-121.23,method='nearest').values
avg2.sel(lat=38.33,lon=-121.23,method='nearest').values
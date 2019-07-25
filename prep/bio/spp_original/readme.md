## Species status

This folder contains all scripts, and most of the data used and generated by each script to develop the species data layer for the OHI Northeast Assessment. Below is a description of the process, including information about each script.

The species status depends upon two pieces of information:  
1. what marine species exist in the US Northeast region?
2. what is their conservation status (e.g. Least Concern, Vulnerable, Critically Endangered)?

We use data from multiple data sources to help answer these questions for all regions in the OHI Northeast assessment. To answer question 1 we use species range maps from the IUCN and more regionally specific range maps developed for the [Northeast Ocean Data Portal](https://www.northeastoceandata.org/). For question 2, we use available conservation status information from [NatureServe](http://www.natureserve.org/) and supplement missing status information with IUCN threat levels.

## What marine species exist in the US Northeast region?

**Get IUCN species range maps**  
We have already downloaded all available IUCN marine species range maps for other projects (see [`spp_risk_dists`](https://github.com/oharac/spp_risk_dists) project). Casey already rasterized these maps for that project and created lookup tables that identify what global cells each species is found in. This is queried in `1_get_ne_iucn_spp.Rmd` which returns `data/1_iucn_spp_in_ne.csv` - a list of all IUCN species with ranges that fall within our Northeast region.

The next step was to rasterize each of those species range maps from IUCN, only for our area. The script `2_rasterize_iucn_spp_shps.Rmd` does just that and saves individual species rasters with values of 1 for presence or NA for absence. These are stored on a server at NCEAS.

**Get Northeast Ocean Data Portal species range maps**  
Where available, we want to use finer scale range map information for species in the US Northeast. The Northeast Ocean Data Portal contains hundreds of species range maps that we prefer to use instead of IUCN range maps. Through our partners we were able to obtain 156 species range maps. The script `3_process_dataportal_spp_maps.Rmd` rasterizes each shapefile and saves to the server.

In total, we have 763 species with range maps for our region. There are 92 species that hae range maps in both databases. We always prioritize the data portal range maps over the IUCN range maps since they were developed specifically for our region of interest.


## What is the conservation status for marine species in the US Northeast?

**Get species conservation status information from NatureServe**  
We access the NatureServe API with the `natserv` R package to retrieve each species conservation status across all scales from global (IUCN) to state level (CT, NY, MA, ME, RI, NH) using the script `4_get_natureserve_data.Rmd`. 328 of our species have an assigned conservation status within NatureServe.

**Get species conservation status information from IUCN**
Since only 328 of our species of interest have a listed status in NatureServe, we wanted to check the IUCN API to see if some species status are captured in this global database but haven't been included in NatureServe. In `5_create_spp_source_status_df.Rmd` we query the IUCN API using the `rredlist` R package and obtain all available status information for the remaining species. This brings us to a total of 712 species with a conservation status.


## Where are these species located within the Northeast?

Now that we have species range maps and their assocatied conservation status information, we need to 
1. link these species maps with our OHI Northeast regions, and 
2. score each species between 0 and 1 based on their conservation status

The script `6_link_spp_cells_rgns.Rmd` iterates through each species range raster, and calculates the amount of area in each OHI region. It also creates a large dataframe that contains each cell ID for each species. These two datasets sit on the server due to large size and will be used to make a map of all species conservation status. This script saves the `spp_rgn_areas` layer used in the toolbox.

After determining the total area of each species range in all OHI regions, we assign each species status a score in `7_assign_spp_status_scores.Rmd`. This script saves the `spp_status_scores` layer used in the toolbox.

## What is the conservation status trend of these species?

Unfortunately we don't have very detailed information on the history of genuine risk for our Northeast species. We still want to assess trends if possible and with the IUCN data we can do that. The script `8_spp_trend_layer.Rmd` creates the `spp_trend` layer used in the toolbox by querying the IUCN API for population trend information (using the `iucn_spp_trend.R` script) and saving a list of all species and their associated trend.

## Additional analyses

- `create_scored_rasters.Rmd` creates individual species rasters with cell values equal to the risk score. These rasters are aggregated to create a single regional species risk (or conservation status) map `data/spp_status_risk.tif`  
- `debug_land_cells.Rmd` was used to determine why we were seeing some species cells show up over land. In the end, these are bird ranges and removed within the analysis.
- `get_spp_historical_status.Rmd` gets previous statuses for IUCN species and saves the file `iucn_spp_historical_status.csv`. This is not currently used in the goal model but could be if we change how we incorporate trend.
- `assign_status_scores.R` gives a value between 0 (no threats, least concern) and 1 (extinct) to each conservation status category, creating `natserv_status_scores.csv`.
- `compare_natserve_w_other_spp_status.R` was used to compare NatureServe data to a dataset provided by Emily. Ultimately it supported our decision to only use NatureServe and IUCN data.

































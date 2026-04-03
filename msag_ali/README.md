# MSAG-GIS Synchronization
Scripts used to synchronize the existing GIS and MSAG data. 


## Contents
* <ins>stType_abbr:</ins> CSV containing the commonly used abbreviations of street type, and their official
postal abbreviations with "Trunk" added. [source](https://pe.usps.com/text/pub28/28apc_002.htm)
* <ins>gis_prep:</ins> Jupyter Notebook with scripts for creating match keys from GIS SSAP data
* <ins>msag_prep:</ins> Jupyter Notebook with scripts for modifying MSAG entries to facilitate matching to GIS
* <ins>msag_sync:</ins>Jupyter Notebook with scripts for matching the GIS SSAP data to MSAG
* <ins>msag_syncToALI:</ins>Jupyter Notebook with scripts for matching the generated MSAG/SSAP sync report to the ALI. Helpful for identifying bottlenecks to edits via Intrado

## Overview
![Overview](https://github.com/Washington-County-GIS/NG911/blob/main/msag_ali/sync_overview.png)


## Notes
In Wisconsin, the statewide method for checking address accuracy between Land Information offices and PSAPs is to compare GIS data to the Master Street Address Guide (MSAG), 
the legacy system used by PSAPs to document roads and address ranges. In counties where the MSAG is still in use, approved users submit edits via a web platform maintained 
by telephone service companies. These updates get pushed to the Automatic Location Identification (ALI). The ALI is not typically edited directly by PSAPs.


Syncronizing GIS data with the MSAG is useful for identifying gaps in the address data on either side, except in cases where the *CAD* does not perfectly match the MSAG. 
Further scripts for comparing GIS data to CAD exports are in development. Keep this in mind as you allocate time and resources to MSAG-GIS synchronization. The end goal 
should be accurate address data in Land Information that matches what PSAPs use for dispatch and other purposes. 

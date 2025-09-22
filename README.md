# Air Quality Sensor Data Repository (AQ-SDR)


This repository holds the code used to pull data, organize, clean, and preprocess from the following sources:

- [KNMI](https://www.daggegevens.knmi.nl/)
- [Samenmeten](https://api-samenmeten.rivm.nl/v1.0/Things)
- [luchtmeetnet, API version](https://api2020.luchtmeetnet.nl/open_api/stations)
- [luchtmeetnet, verified](https://data.rivm.nl/data/luchtmeetnet/)
- [Sensor Community](https://archive.sensor.community/)
- [Location Aware Sensor System - Taiwan](https://lass-net.org/)
- [Ministry of Environment - Taiwan](https://data.moenv.gov.tw/en/)


Luchtmeetnet API is the one available to pull for the public. The verified version is the one verified by the government and reported to the European Union. For the use as a reference data, we highly recommend you use the official verified version.

We also provide a large chunk of the raw data provided through the following link:

```
insert figshare link here.
```

At the end of this readme file, you will see a description of the dataset tools and some explanation.

Here, we will explain how to prepare to use the data for ML models as we have done in our paper [Veli](https://github.com/YahiDar/Veli).


## Preprocessing

To abide by the licensing provided by each data source, we provide the raw data through the figshare link posted above. 

I added a requirements.txt file that captures the libraries, not ALL of them are required, this is just the latest stage of the environment.

We provide a collection of Python scripts that does all required preprocessing for this dataset. You only need to run the ```run_data_preperation.sh``` shell file to run all these scripts.

The script will take a long time, upwards of 10 hours to prepare ALL the data. you will need at least 32 GB of RAM and a total of 75 GB of storage.

Due to the licesning, we cannot publish the processed data. 
These scripts do:
- Reorganize the data
- Resample the data hourly
- Create metadata
- Apply DBSCAN on all files to ensure validity
- Create a complete metadata (available as a solo json file in the `/metadata/` folder)
- Creates the dataloaders required for the Veli model





The python scripts are split into:

- Downloading and pulling
- Reorganizing (if required)
- Preprocessing (dbscan and outlier removal)






## utils

General files to support the main scripts.

## KNMI

Scripts: Data requests `data_requests_knmi.py`.

Pulls the data and reorganizes it in the same file. The data is verified and pulled from reliable weather stations, so no preprocessing is done.


## Samenmeten

Scripts: Data requests `data_requests_samenmeten.py`, Data preprocessing `preprocess_samenmeten.py`, and geo-location fixing `fix_coordinates_samenmeten.py`

The data has 'historical locations' for certain sensors - implying mobility. Sometimes this is a gps error in the decimals, which is disregarded. Other times it is significant (larger than 2km movement), so every station with larger separation like this is split into multiple stations based on the duration it spends there. Any station with multiple (more than 5) movements is disregarded (very few stations, less than 20)

## Luchtmeetnet
There are two variants of these data - the api version and the csv files version. The CSV are confirmed and verified by the publishing entity (Amsterdam Government) and is used to report to the European Union, hence it is the highest level of accuracy available to us.

### API Version

Scripts: Data reorganization ```data_requests_luchtmeetnet_api.py```, Data preprocessing (not implemented yet) `preprocess_luchtmeetnet_api.py`


Pulls the data and reorganizes it in the same file. Slow and incomplete - still have not implemented preprocessing since pulling is not over.

### Verified Version

Scripts: Data reorganization `organize_luchtmeetnet_verified.py`, Data preprocessing `preprocess_luchtmeetnet_verified.py`

You can download the data separately - no script to be provided here.

Reorganizes the data since it comes in an extremely weird format. The data is verified and pulled from reliable weather stations, but there were noticeable error so preprocessing with loose constrainst are implemented.

## Sensor Community (SenCom)

Scripts: Data requests `data_requests_sencom.py`, Data preprocessing `preprocess_sencom.py`

Very large dataset, pulled and limited only to Netherlands, Belgium, and Germany. More than 2 TB of data.


## Out of Distribution data (Taiwan)
All out of distribution data was downloaded from [here](https://history.colife.org.tw/#/).

We provide the identical files in our comprehensive dataset with no code to pull them through any API.


# AQPreprocessing


This repository holds the code used to pull data, organize, clean, and preprocess from the following sources:

- [KNMI](https://www.daggegevens.knmi.nl/)
- [Samenmeten](https://api-samenmeten.rivm.nl/v1.0/Things)
- [luchtmeetnet, API version](https://api2020.luchtmeetnet.nl/open_api/stations)
- [luchtmeetnet, verified](https://data.rivm.nl/data/luchtmeetnet/)
- [Sensor Community](https://archive.sensor.community/)

Luchtmeetnet API is the one available to pull for the public. The verified version is the one verified by the government and reported to the European Union.

The python scripts are split into:

- Downloading and pulling
- Reorganizing (if required)
- Preprocessing (dbscan and outlier removal)


Note that these scripts are not fully parametrized, be careful with root directories and other directories - make sure that they match your setup.

I added a requirements.txt file that captures the libraries, not ALL of them are required, this is just the latest stage of the environment.

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



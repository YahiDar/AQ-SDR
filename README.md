# Air Quality Sensor Data Repository (AQ-SDR)
If you are using this dataset for the Veli model, make sure to check the other repository for the model code and running at: [Veli](https://github.com/YahiDar/Veli).

Also, please do cite our paper using:

```

```


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

[Download the dataset from figshare](https://figshare.com/s/3d76b6c57ed6913bb8fe)

At the end of this readme file, you will see a description of the dataset tools and some explanation.

The bash script will produce a dataset ready for running through the following directory trees:

EU Data:

```
final_dir_eudata
├── crowd_stations_root_dbscan
├── data
├── KNMI
├── luchtmeetnet_csvs
├── luchtmeetnet_csvs_dbscan
├── lucht_root_dbscan
├── metadata
├── sencom_final_root_dbscan
└── final_dataset
   ├── prepared_lcs_bulk
   └── pre_prepared_datasets_unfiltered


```
Taiwan Data:
```
final_dir_ood
├── downloaded_lcs
├── downloaded_ref
├── lcs_data_dbscan
├── metadata
├── ref_data_dbscan
└── final_dataset
    ├── ood_data_bulk
    └── prepared_ood_datasets

```


# Environment Setup

All packages and dependencies used are available via two options:
- `requirements.txt`
- `pyproject.toml` (Poetry)

This guide assumes you are using a bash-based tool (linux/mac). These are the exact same instructions and packaging files as the ones in the [Veli](https://github.com/YahiDar/Veli) repository.
You can install all necessary packages **EXACTLY** following our work using the following commands:



## 1. Using Python Virtual Environment

### a) With `requirements.txt`
```bash
# Create and activate a virtual environment
python3 -m venv veli
source veli/bin/activate   # On Windows: veli\Scripts\activate
pip install -r requirements.txt
```

### b) With `poetry.toml`
Install poetry if not installed (feel free to create a virtual environment beforehand)
```bash
pip install poetry
```

Create and activate the virtual environment + install deps. Also activate the shell.
```bash
poetry install

poetry shell
```

### c) Using Conda

```bash
conda create -n veli python=3.10

conda activate veli

pip install -r requirements.txt
```

## Preprocessing

To abide by the licensing provided by each data source, we provide the raw data through the figshare link posted above. 

Due to the licesning, we cannot publish the processed data. 
These scripts do:
- Reorganize the data
- Resample the data hourly
- Create metadata
- Apply DBSCAN on all files to ensure validity
- Create a complete metadata (available as a solo json file in the `/metadata/` folder)
- Creates the dataloaders required for the Veli model


### Running the Bash Script:

We provide a collection of Python scripts that does all required preprocessing for this dataset. You only need to run the ```run_data_preperation.sh``` shell file to run all these scripts.

The script will take a long time, upwards of 10 hours to prepare ALL the data. you will need at least 32 GB of RAM and a total of 75 GB of storage.

The bash scripts run the two python files `prepare_all_data.py` which prepares the data for the EU region (in-distribution), and `prepare_taiwan_data.py` which prepares the data for the Taiwanese region (out-of-distribution).
The python scripts are modular but I set the variables internally. Feel free to change them and play with them as you wish. I will however not be addressing issues in regards to parameters that I have not supported :)!

It will create a log file in this directory called `run_data_preperation.log`.

These are sample arguments for the python scripts:

### EU data


```bash
python -u prepare_all_data.py  --eu_data "/path/to/eu_data"  --final_dir "/path/to/final/data" --dummy_holder "/path/to/dummy_holder" 
  
```
The path to eu_data should contain the following directories:

```
required_subdirs = [
    "crowd_stations_root",
    "KNMI",
    "luchtmeetnet_csvs",
    "lucht_root",
    "sencom_hourly"
]
```

This will automatically delete the 'dummy_holder' directory after you are done since it is not needed. If you wish to observe the whole process of preprocessing, feel free to activate the argument `--keep_dummy` which will not delete them. NOTE: this requires an additional 100GB of storage.

### Taiwan data
```bash
python -u prepare_taiwan_data.py --operation_root "/path/to/taiwan_raw/"
```
The path to taiwan data should contain the following directories:

```
required_subdirs = [
    "downloaded_ref",
    "downloaded_lcs"
]
```

Similar to EU data. Additionally, you can activiate the argument `--keep_dummy` to keep the dummy folders.

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


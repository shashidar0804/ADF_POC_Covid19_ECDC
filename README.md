# ADF_ETL_POC_Covid19_ECDC
This is a complete ETL Project done on ADF with Reporting on PowerBI using the Europe Covid19 Data. The data has been taken from EuroStat and European Centre for Disease Prevention and Control (ECDC) wesbsites.
Below are the types of datasets used for this project. Some of them are the files directly fed to the data pipelines and some of them are the URL links added in a lookup file and passed as an input to the data pipelines:

| DataSet | Description | 
|----------|---------- |
| population_by_age.tsv.gz (EuroStat/Blob) | This is a zipped file that contains the population data of countries. |
| cases_deaths.csv (ECDC) | This csv contains the number emerging Covid Cases and Deaths followed by the each day. |
| hospital_admissions.csv (ECDC) | This csv file contains the Daily Hospital Admissions, Daily ICU admissions, Weekly Hospital Admissions per 100k, Weekly ICU Admissions per 100k. |
| lookups (Blob) | Other miscellaneous files like Calendar Lookup/dim and Country lookup/dim files used. |

## Solution Architecture

Below is the solution architecture of the ETL Project End to End

![project_solution_architecture](https://github.com/user-attachments/assets/9ff2c862-0cc5-4255-9e45-6627b032a57e)

### Step1:
Firstly i have used a manual uploading of a sample downloaded zipped file from ECDC website and uploaded that to Blob storage. Then i have used Azure Data Factory Copy activity which unzips the zip file and copies that to Azure Data lake storage (ADLS gen2)


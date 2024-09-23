# ADF_ETL_POC_Covid19_ECDC
This is a complete ETL Project done on ADF with Reporting on PowerBI using the Europe Covid19 Data. The data has been taken from EuroStat and European Centre for Disease Prevention and Control (ECDC) wesbsites.
Below are the types of datasets used for this project. Some of them are the files directly fed to the data pipelines and some of them are the URL links added in a lookup file and passed as an input to the data pipelines:

| DataSet | Description | 
|----------|---------- |
| population_by_age.tsv.gz (EuroStat/Blob) | This is a zipped file that contains the population data of countries. |
| cases_deaths.csv (ECDC) | This csv contains the number emerging Covid Cases and Deaths followed by the each day. |
| hospital_admissions.csv (ECDC) | This csv file contains the Daily Hospital Admissions, Daily ICU admissions, Weekly Hospital Admissions per 100k, Weekly ICU Admissions per 100k. |
| lookups (Blob) | Other miscellaneous files like Calendar Lookup/dim and Country lookup/dim files used. |

Below are the Azure Services that i have created and tagged under my dashboard for this ETL POC

![image](https://github.com/user-attachments/assets/ceabee6c-291c-44f8-8b1b-6e52971995b4)


## Solution Architecture

Below is the solution architecture of the ETL Project End to End

![project_solution_architecture](https://github.com/user-attachments/assets/9ff2c862-0cc5-4255-9e45-6627b032a57e)

### Step 1:
Firstly i have used a manual uploading of a sample downloaded zipped file from ECDC website and uploaded that to Blob storage. Then i have used Azure Data Factory Copy activity which unzips the zip file and copies that to Azure Data lake storage (ADLS gen2). I have used linked service(ls_ablob_covidreportingsa) to connect to blob storage(covidreportingsa). I have also used a linked service(ls_adls_covidreportingdl) to connect to data lake storage(covidreportingdlv02)

![image](https://github.com/user-attachments/assets/6132774c-2641-4e7f-b07f-f4f968a71020)

### Step 2:
Now I have taken 2 datasets i.e cases_and_deaths, hospial_admissions which are uploaded in the GitHub repository under Covid19-Europe-Project/main-csv-data-files (These are the same files available in ECDC website). For connecting to these files in ADF i have used  https: Linked Service(ls_http_opendata_ecdc_europa_eu) and gave my base url name to it. Since I wanted to create ingest both the datasets into ADLS gen2 at a time, I created a json file with  files and created parameterized datset and with the help of Lookup acivity and ForEach activity and i was able to successfully ingest the data into the ADLS gen2.

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

### Step 2 (Data Ingestion):
Now I have taken 2 datasets i.e cases_and_deaths, hospial_admissions which are uploaded in the GitHub repository under Covid19-Europe-Project/main-csv-data-files (These are the same files available in ECDC website). For connecting to these files in ADF i have used  https: Linked Service(ls_http_opendata_ecdc_europa_eu) and gave my base url name to it. Since I wanted to create ingest both the datasets into ADLS gen2 at a time, I created a json file with  files and created parameterized datset and with the help of Lookup acivity and ForEach activity and i was able to successfully ingest the data into the ADLS gen2.

![image](https://github.com/user-attachments/assets/0f08f8d5-057d-473e-9f8a-86f765e3a27b)

### Step 3 (Processing):
Once right after the required data ingested into the ADLS gen2 storage, I have created the dataFlows for both the datasets to process further with the transformations and load into the sink which is SQL Server. Now the DataFow will help us to create the transformations according to the project requirement. Once after the dataFlows build, I created the the pipelines for all of those 2 datasets which redirects to the sink.



#### Cases and Deaths Dataset Transformations:
#### Requirement:
To read the Cases and Deaths file and filter only Europe data. In the file there are Confirmed cases count and the deaths count populated in the same column. The requirement is to pivot the the counts into two columns based on the indicator column values (Confirmed cases and deaths)

Below are the transformations in the dataflow

![image](https://github.com/user-attachments/assets/07007c85-ced3-48cb-acb0-b1ca9f1fa5f8)

1. Reading the raw source from the data lake
2. Used Filter transormation to filter out only Europe records (continent == 'Europe' && not(isNull(country_code)))
3. Post selecting the required columns using the Select transformation added the Pivot transformation based on the pivot key "indicator" column and the values as "Confirmed cases" and "deaths" and the pivoted column aggregate transformation is sum(daily_count). Also used group by all the other columns
4. Did a lookup to the country lookup file to get the country 2 digit and 3 digit codes
5. Post selecting the required columns using the select transormation , used the sink transformation to populate the data to Data Lake as a processed file

#### Hospital and Admission Transformations:
#### Requirement:
To read the Hospitals and admissions csv file and lookup country file to get the 2 digit and 3 digit country codes and values. Split the file into two parts. One for Weekly and the other for Daily based on the indicator field. Pivot both daily and weekly files based on the indicator value and get the necessary counts. Sort the weekly data based on the reported_yer_week desc and Country asc and populate the data to datalake

Below are the transformations done:

![image](https://github.com/user-attachments/assets/dead25d0-d732-40d6-90a0-b0d75983fd77)

Similar transformations were done like the cases and deaths except below extra ones:
1. Split Transformations to split the weekly data with daily data (indicator == "Weekly new hospital admissions per 100k" || indicator == "Weekly new ICU admissions per 100k"). The data that doesnt match the condfition goes to daily file
2. Before joining to the Dim date source file there is the aggregation done on the file to get the week start and end date for every week with group by (year+"-W"+lpad(week_of_year,2,'0')) and week start as min(date) and week end date as max(date)
3. On Weekly and Daily splits similar kinds of Pivot transformations are applied to get the necessary counts and finally sorting is done and populated to the Sink which Data Lake as a processed file

### Step 4 (Sqlize)
Created two pipelines (pl_sqlize_cases_and_deaths and pl_sqlize_hospitals_admissions) to populate data into SQL Server. For this created a copy activity to copy the data from the data lake to SQL Server. For the connection to SQL server create a linked service(ls_sql_covid_db) with sql authentication connection type for the server already created and available in the Dashboard.

![image](https://github.com/user-attachments/assets/b2bbc1ff-22c5-4e9f-9253-7b50926caa91)

![image](https://github.com/user-attachments/assets/76e52618-e63b-4247-8d68-4ec8e22fd23b)


### Step 5 (Visualization):
After we did transformation according to the project requirement, the data is in the SQL database, so that Data Analysts can query data directly from the database by using PowerBi/Tableau or any other data visualization tool for analysis purpose. Data visualization part is not done in this POC

![image](https://github.com/user-attachments/assets/8527f888-fb02-4785-b5d5-1f7030decab8)

![image](https://github.com/user-attachments/assets/4b6433c4-7712-4cb2-9463-ac04d5d70d3a)


Finally created triggers (Tumbling window trigger) to trigger for the particular point in time for every 

![image](https://github.com/user-attachments/assets/eda80e7b-eb7e-4b48-a7e1-c135a32dc0a8)





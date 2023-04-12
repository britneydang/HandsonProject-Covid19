# Hands-on Project:
Description: This project is about creating everything from scratch using all technologies/tools that are available in Microsoft Azure Data Factory using Covid-19 datasets

Data Resources:
Use COVID-19 data from the EU Center for Disease Prevention and Control website (ECDC).
- ECDC Website for Covid-19 Data - https://www.ecdc.europa.eu/en/covid-19/data
- Euro Stat Website for Population Data - https://ec.europa.eu/eurostat/estat-navtree-portlet-prod/BulkDownloadListing?file=data/tps00010.tsv.gz

Solution Architect:
- I will use the HTTP Connector within the ADF to get the COVID-19 data (data set #1) and (as a second connector) keep the Population data (data set #2) in an Azure storage account and ingest from there.
- Both data sets will be ingested into the Azure Data Lake Storage Gen2.
- I will Data Flows (transformation tool) within Data Factory.
- All transformed data will be moved to Azure Data Lake Storage Gen2 for Machine Learning use.
- I also push the subset of the required data to the SQL database so it can be used for reporting.
- Visualization report such as PowerBI can be built from this data.

Technologies:
- Use Azure Data Factory for all data integration and orchestration.
- Transformation tools: Data Flows (simple transformation, code-free)
- Storage solutions: Azure Blob Storage, Azure Data Lake Storage Gen2 (can add the big data solutions like Hadoop, Synapse Analytics), Azure SQL Database for data warehouse
- Reporting tool: PowerBI

My steps:
1. Environment Setting Up:
- Create an Azure account and log in https://portal.azure.com/#home.
- Create Azure Data Factory resource via Azure portal: Create a resource -> Marketplace -> Integration -> DF. After create DF and deployment is completed, Go to Resource. Click on Launch studio.

![image](https://user-images.githubusercontent.com/110323703/207212129-9b1dd4a4-38dd-44d3-8a90-3c63199c5ea8.png)
https://portal.azure.com/#@britneydang111gmail.onmicrosoft.com/resource/subscriptions/111234fb-c6b9-407a-835e-3aa626721019/resourceGroups/COVID19-data-BritneyD/providers/Microsoft.DataFactory/factories/COVID19-reporting-ADF-BritneyD/overview

- Create a Storage Account: Create a resource -> search Storage Account, leave everything default for now, Replication = Locally-Redundant Storage. When deployment is completed, Go to Resource.

![image](https://user-images.githubusercontent.com/110323703/207213881-d462b69f-e6da-41ec-beee-9aadd327cae5.png)
https://portal.azure.com/#@britneydang111gmail.onmicrosoft.com/resource/subscriptions/111234fb-c6b9-407a-835e-3aa626721019/resourcegroups/COVID19-data-BritneyD/providers/Microsoft.Storage/storageAccounts/covid19reportingbritneyd/overview

- Create an Azure Data Lake Gen2: Create a resource -> search Storage Account, Replication = Locally-Redundant Storage, turn off Soft Delete, enable Data Lake Storage Gen 2 in Advanced (this feature is the only difference compared to other storage account).

![image](https://user-images.githubusercontent.com/110323703/207215273-c5b51ccd-39fc-4a78-b38c-1189d549e373.png)
https://portal.azure.com/#@britneydang111gmail.onmicrosoft.com/resource/subscriptions/111234fb-c6b9-407a-835e-3aa626721019/resourcegroups/COVID19-data-BritneyD/providers/Microsoft.Storage/storageAccounts/covid19datalakebritneyd/overview

- Create an Azure SQL Database: Create a resource -> Databases -> SQL Databse, create new Server -> authentication method = SQL authentication and set admin: britney/password: BDang1991, Configure database to Basic for more affordable price per month, Connectivity method = public endpoint and allow Azure service access to this server, Replication = Locally-Redundant Storage. When deployment is completed, Go to Resource.

![image](https://user-images.githubusercontent.com/110323703/207219231-25fdc375-2dac-4b34-b9f7-fba6e90dc911.png)
https://portal.azure.com/#@britneydang111gmail.onmicrosoft.com/resource/subscriptions/111234fb-c6b9-407a-835e-3aa626721019/resourceGroups/COVID19-data-BritneyD/providers/Microsoft.Sql/servers/covid19serverbritneyd/databases/covid19databasebritneyd/overview

In order to create a new table, I need to change Firewall settings, add my own IP and save. Go to Query Editor on right side, log in use admin/password.
- Launch Azure Data Studion: Copy/paste server name from the newly create database into New connection server, enter other info and connect.

2. Data Ingestion - Azure Blob Storage - data set #2 
- I will ingest the Population data set from Azure Blob Storage to the data lake.
- There are 3 areas: Source, Pipeline, Sink
  - Source: storage account, container, and the original file that will be copied across. I already have had storage account, need to create a container and get the data set #2.
  - Pipeline: 6 DF components need to be created: Linked Services for Source, Source Data Set, Linked Service for Sink, Sink Data Set, Pipeline, Copy Activity.
  - Sink: storage account, container (raw), and the file itself that would be copied from Source.

![image](https://user-images.githubusercontent.com/110323703/207225866-bcf7ec29-0af1-45ce-9be0-b6a5dd5f04a7.png)
  - Create file container and upload the file into the Blob Storage Account. Go to previously created Storage Account -> Storage Browser -> Blob container -> Add Container -> private -> create -> click on newly created container and select data set #2 -> upload.
  - Create container on the data lake storage. Go to previously created Data Lake Storage Account -> Storage Browser -> Blob container -> Add Container -> name "raw". 
  - Create 2 Linked Services: Go to ADF -> Manage -> Linked Service -> New -> choose Azure Blob storage or Azure Data Lake Storage Gen2 -> add info -> create

![image](https://user-images.githubusercontent.com/110323703/207411453-17c76dc5-e3a4-4551-bf5c-488e603a4f55.png)
  - Create 2 Datasets -> ADF -> Author -> Dataset -> New Dataset -> choose Azure Blob storage (DelimitedText) / Azure Data Lake Storage Gen2 -> set properties: set file path, first row as header, NONE import schema -> OK -> Compression type: NONE / NONE (if it is a zip file, can select gzip), Column delimiter: tab / tab. PUBLISH ALL -> publish.
  - Create ADF pipeline: ADF -> new pipeline -> add info -> start to draw a flow in the blank area  
    - Click Validate ALL -> see no error -> DEBUG -> after succeeded -> PUBLISH ALL to save the pipeline
    - Control Flow Activities: Drag GET METADATA, in properties: update name, select original dataset, Field list: add a specific column names as needed for IF conditon (filter). Drag IF CONDITION in, connect GET METADATA to it, in the properties below, General: update name, Activities: Add dynamic content, use the pipeline expression builder. IF TRUE, the example below mean: if the chosen column "Count" equals to 13, then it is true. 

![image](https://user-images.githubusercontent.com/110323703/207449393-95b9a356-94aa-4333-82d6-70c95283bfdb.png)
IF FALSE, edit -> Drag WEB, input for properties, can use a dummy URL so it will show an failed error.
    - Drag COPY data in -> in the properties below, General: update name, timeout, Source: select original file, Sink: select target file, Mapping: no custom mapping needed if I simply move the whole file from 1 location to another. If the IF CONDITION is true, the COPY activity will be called. If the IF CONDITION is false, the COPY activity will not be called and I can set up an email sending out for error messages.
    - Drag VALIDATION in, in properties: update name (name:" Check if file exists"), select original dataset, edit timeout (check if file is arrived within ... hour, otherwise it will fail), sleep (how often do I want to check the file), minimum size. Make GET METADATA dependent on VALIDATION by connecting the 2 activities. GET METADATA will only run if the VALIDATION succeeds. DEBUG.
    
![image](https://user-images.githubusercontent.com/110323703/207456388-d76fa77d-68ee-425c-b86f-59facdea0234.png)
    - Delete the source file on successful copy (acting like a move, but ADF doesnt have MOVE activity): Add a DELETE activity after the COPY activity. Under properties, select original dataset, unable logging (enable if I need to keep track a list of deleted files), connect them. PUBLISH ALL.

![image](https://user-images.githubusercontent.com/110323703/207461737-7b1621c0-b903-4f83-9e6d-4dff23ae3302.png)
  - My pipeline is doing a copy of a file from a Blob storage to an Azure Data Lake and I expect the file in the Blob Storage to arrive once every day. When the file is picked up, go back and delete the file (this acts like a move). Now I need to decide which trigger type. 3 types: Schedule, Tumbling Window, Event. It can be Schedule because I am expecting the file to arrive every day at a certain time. It can be Event because I have the file deleting at the end of the processing and I want to process the next file as soon as it arrives. Create trigger in ADF: ADF -> Manage -> New trigger -> name "tr_ingest_population" -> Event

![image](https://user-images.githubusercontent.com/110323703/210422250-8f911026-f03d-44f0-a2e9-3c0155b7435f.png)

  - After creating a trigger, I need to attach it to a pipeline. Go to the pipeline pl_ingest_population_dataset2 -> Add trigger -> New/Edit -> Select tr_ingest_population_dataset2 -> OK -> PUBLISH ALL. To test run the trigger, go to the Azure Storage Explorer to reupload the population_dataset2.tsv file, go to Monitor Trigger Runs to refresh and view. 

![image](https://user-images.githubusercontent.com/110323703/210429361-d73d6cf3-4952-4027-8ede-8b51d4ba015c.png)

2. Data Ingestion from HTTP - data set #1
- I will ingest the data sets from HTTP (https://www.ecdc.europa.eu/en/covid-19/data) to Azure Data Lake. For the purpose of keeping my project data unchanged, I have saved the a;; the ECDC data into my git hub. The URL Base will be github website instead of ECDC website.
- ADF -> Manage -> New Linked Services -> Select HTTP -> Leave as default for integration runtime, and copy paste the URL into BASE URL= https://github.com/, Authentication = Anonymous -> Test connection -> Create
  - Create the Source dataset - HTTP dataset: Author -> Dataset -> New -> HTTP/csv -> select Link Services and add Relative URL = britneydang/HandsonProject-Covid19/blob/main/cases_deaths.csv, First row as Header -> OK
  - Create the Sink dataset: Author -> Dataset -> New -> Azure Data Lake Storage Gen2/csv, choose linked services, add file path and csv file name, Import schema NONE -> OK
  - Create the Pipeline: Author -> New -> Drag and Drop to create the pipeline. Under section, in General tab, rename. In Source tab, select source dataset http. In Sink, select sink dataset datalake. Click DEBUG so the file from the URL will get copied to the data lake. After it succeeded. PUBLISH ALL.

![image](https://user-images.githubusercontent.com/110323703/210641197-bba70602-ae45-4baf-9c36-16703f9fde55.png)
  - To avoid creating many components, I want to use the same existing pipeline (pipeline already create for cases deaths dataset) to pass many different file names. ADF has an ability to do so in the form of parameters and variables. Pipeline will be more dynamic that it can deal with different file everytime it is called.
    - Parameters are external values passed into pipelines, datasets or linked services. The value cannot be changed inside a pipeline.
    - Variables are internal values set inside a pipeline. The value can be changed inside the pipeline using Set Variable or Append Variable Activity.
  - For Source, BASE URL (https://github.com/) is the same, only change the Relative URL for different file name (britneydang/HandsonProject-Covid19/blob/main/... .csv). Same for Sink, in the data lake storage, container (rawpopulation) and folder (ECDC) are the same, just the file name is different (rawpopulation/ecdc/... .csv)
  - For the Source dataset, I want to parameterize the Relative URL: Author -> select an existing dataset (http) -> in Parameters tab -> New -> name = RelativeURL, type = string -> in Connection tab -> remove existing relative URL and click on Add dynamic content. Dynamic name will be replaced.

![image](https://user-images.githubusercontent.com/110323703/210646015-a79067cd-a5ea-4596-b519-3d37a6980079.png)
  - For the Sink dataset, I want to parameterize the file name: Author -> select an existing dataset (datalake) -> in Parameters tab -> New -> name = FileName, type = string -> in Connection tab -> remove existing FileName and click on Add dynamic content. Dynamic name will be replaced.

![image](https://user-images.githubusercontent.com/110323703/210646913-1929c041-c180-4cae-862d-5720e66675b2.png)
  - After both are parameterized, I need to change the pipeline. 
  - ** Option 1 ** Use variables within pipeline to test the changes in the datasets. I want to create a few variables within the pipeline and then pass those variables to the datasets to test the changes that I just made: Author -> select previous pipeline that created for cases death dataset -> in Variables tab -> New
    - name1 = sourceRelativeURL, type1 = string, default value1 = britneydang/HandsonProject-Covid19/blob/main/hospital_admissions.csv
    - name2 = sinkFileName, type2 = string, default value2 = hospital_admissions.csv
  - I want to map these 2 variables to the parameters. Click on Copy activity:
    - In the Source tab, at RelativeURL, add dynamic content sourceRelativeURL
    - In the Sink tab, I need to pass in the file name. at FileName, add dynamic content sinkFileName
![image](https://user-images.githubusercontent.com/110323703/210650880-d7e6b367-c6e3-4d93-9e56-e49d00bc8069.png)
![image](https://user-images.githubusercontent.com/110323703/210651157-80fa4bcc-9b28-4883-a301-740dcd264eb5.png)

  - DEBUG. It should copy the new data (hospital admissions) from the URL into my data lake.
  - ** Option 2 ** Parameterize the pipeline. I want the pipeline is truly generic so I can pass on the parameters from the trigger. Delete previous created variables in option 1. Create new Parameters: sourceRelativeURL and sinkFileName. Click on COPY activity -> add dynamic content for RelativeURL (Source tab) and FileName (Sink tab)

![image](https://user-images.githubusercontent.com/110323703/210654185-de6cb4ca-b37c-4729-be9d-dfbb0fab42e7.png)
![image](https://user-images.githubusercontent.com/110323703/210654291-a3c6b4e3-9a85-4906-b086-fcf698d430d2.png)
  - Update Source and Sink datasets name, Pipeline name, COPY activity name into a generic name.
  - DEBUG. In pipeline run , it asks for the value, need to input a specific RelativeURL and FileName. In this case, they will be:
    - sourceRelativeURL: britneydang/HandsonProject-Covid19/blob/main/hospital_admissions.csv
    - sinkFileName: hospital_admissions.csv
  - ** Option A ** After succeeded, I will create a trigger to keep the data updated. In this case, I will use Schedule trigger because the file in the URL will always exist, ECDC website update their data anytime they want and I wont look at slices of data (I want to get the data at a certain point): Manage -> Triggers -> New -> add info -> Ok 
  - Need to attach trigger to the pipeline befor PUBLISH. Go to the pipeline -> Add trigger -> New/Edit -> Choose the newly created trigger-> Ok -> input the Value:
    - sourceRelativeURL: britneydang/HandsonProject-Covid19/blob/main/hospital_admissions.csv
    - sinkFileName: hospital_admissions.csv

![image](https://user-images.githubusercontent.com/110323703/210661594-e13adbca-fd6a-4dc1-af6b-40e2f6dfa666.png)

![image](https://user-images.githubusercontent.com/110323703/210663222-1b080e26-326c-4aaf-b39c-cfc9a9b96e78.png)
  - Parameterize Linked Services: I need to parameterize linked services in case the BASE URLs (website links) are different. ADF -> Manage -> Linked Services -> click on http -> remove old name Base URL, in Parameters add new "sourceBaseURL" -> click adding dynamic content and select newly added parameters -> Apply. Now I need to create a parameter in existing http dataset: Author -> datasets -> click ds http -> in parameters -> add new -> input BaseURL -> go to Connection tab, add dynamic content -> select BaseURL. I need to make a corresponding change to the pipeline: go to the ingest ecdc pipeline -> in Parameters, add new "sourceBaseURL". I need to pass that onto a source dataset: click the COPY activity, in Source tab, add dynamic content for BaseURL. Click DEBUG and enter info into parameters like below as example. After succeed, go to Azure Storage Explorer to see if the new file in in the data lake.
    - sourceBaseURL: https://github.com/
    - sourceRelativeURL: britneydang/HandsonProject-Covid19/blob/main/country_response.csv
    - sinkFileName: country_response.csv
  - ** Option B ** Instead of creating a trigger to copy from sourceRelativeURL to sinkFileName for each dataset (I have total 4 datasets), I want to have a pipeline that gets information about the Source and the Sink from somewhere and then copies all the data required. Later on, I can invoke that one pipeline with one trigger. To do so, I can use LookUp and ForEach activities in ADF.
    - LooKUp Activity: can retrieve a dataset from any of the Azure DF supported data sources. Its output can be used in a ForEach Activity if it's an array of attributes.
    - ForEach Activity: get the information from the LookUp activity. ForEach activity will go through a list of file names or item names that I've got within a file  and then invoke the COPY Activity or any other Activity for that matter. In this case, I will create a JSON file for needed information (info about 4 datasets) name "ecdc_file_list.json" and I will upload the file into Azure Blob storage so ADF can read from it. If I have more datasets, I dont need to make any change to the ADF, I will only need to update the JSON file.
  - Azure Storage Explorer -> create new blob container name "configs" -> upload the JSON file
  - ADF -> Author -> create new dataset -> Azure Blob storage/JSON -> make sure to select the JSON file for file path -> Linked service is the blob storage ls -> Ok -> this list is created. PUBLISH.
  - ADF -> Author -> ecdc pipeline -> create Lookup activity by dragging it to the pipeline -> in Setting tab, select source dataset, uncheck First Row Only -> create ForEach activity and make it dependent on Lookup activity -> in Setting tab, in Items add dynamic content -> in Activity output, click newly created Lookup activity name + ".value" -> copy the existing COPY Data activity and put it inside the ForEach activity and delete the OG COPY Data activity -> Go to COPY Data activity, in tab Source, add dynamic content for Relative URL and BaseURL -> select under ForEach Iterator and add ".sourceRelativeURL", ".sourceBaseURL". In tab Sink, add dynamic content for FileName -> select under ForEach Iterator and add .sinkFileName. Click on VALIDATE. I need to link my Lookup activity to the JSON file: Author -> dataset -> choose correct file in file path. DEBUG without any input for parameters.
  - Check if 4 dataset files are in the data lake in Azure Storage Explorer
![image](https://user-images.githubusercontent.com/110323703/211281120-780cecf4-3834-483b-9a76-402c7b45ee1d.png)

  - I need to create a trigger that will copy all of the data (4 datasets): Manage -> triggers -> type = Schedule, recurrent = ..., start trigger on creation. Then I need to attach the trigger to the pipeline: Author -> pipeline -> New/Edit -> select the corresponding pipeline -> Ok. Make sure that the new trigger's parameters tab is empty. PUBLISH ALL.
![image](https://user-images.githubusercontent.com/110323703/211285816-e2caa779-5a33-496b-893a-7c0af790b36a.png)

3. Data Transformation - Data Flows in Azure Data Factory
After ingesting data from various sources (ECDC data from website and population data from the Blob Storage) into the Data Lake, now I need to transform the data by using Data Flow (in ADF). There are many types of transformation that can be created with a data flow: source transformation, filter transformation, select transformation, pivot transformation, lookup transformation, sink transformation.
- Transform cases_deaths dataset: Go to Azure Storage Explorer -> datalake -> open the new file cases_deaths.csv in excel -> I need to decide how I want to transform the original data depending on the need of reporting. Below is how the original file looks like. 

![image](https://user-images.githubusercontent.com/110323703/211485728-df606f25-5b31-42a1-8991-23516a786261.png)

Notes of transformations that I want to make:
- Change #1: Filter column continent = "Europe" and remove continent column
- Change #2: Make standard for column country_code into 2 digits across all of the files (For the country_code that has 3 digits, use an external file with lookup function to lookup for its corresponding 2 digits). Keep the country_code 3 digits in there. File name country_lookup.csv. Upload this lookup file into the storage account: Go to Storage Explorer -> ADLS Gen2 -> Blob Container -> create new blob name "lookupsupportfile" and upload the csv file.
- Change #3: Use 2 columns indicator and daily_count to pivot out 2 different columns cases_count and deaths_count
- Change #4: Change column name from date into reported date
- Change #5: Drop column rate_14_day
- Keep columns country, population, source the same

I will start to create a data flow: ADF -> Author -> click turn on Dataflow Debug (Warning notes: Turn on/off the Dataflow DEBUG when it is not in use, if not it will be costly), leave as is AutoResolveIntegrationRuntime (in case I want a bigger cluster for my debug, I can create a new integration runtime) -> Data Flows -> new Mapping Data flow -> dataflow -> name dataflow -> Now I need to create a SOURCE. In a data flow, which is called stream, is started with at least one source and ended with a sink. ALL Tranformations will be performed based on the list of changes above:
- Source Transformation: Need to create a source -> click Add Source. Under Source settings tab -> add Stream name, Source type = Dataset -> add New dataset -> ADLS Gen2 -> DelimitedText -> name df, select corresponding data lake linked service, add file path, select cases_deaths.csv file, select First Row as header and From connection/store -> Ok. Test connection. Next is the Options:
  - Allow schema drift: its useful when I expect my source data to change often. If there is a new column being added to the dataset, data flow will flow the new column through to the sink without me having to make any code changes.If I don't select, it will be the fixed schema that I have specified. 
  - Infer drifted column types: For the new column or any changes being made to the source data, select this option if I want data flow to infer the column types. If I don't select, everything that being added or inferred will have type = string.
  - Validate schema: if my current data doesn't satisfy the dataset schema specifed, it will fail.
  - Other - Skip line count: if I specify number of records to skip within the file, then it will do it.
  - Other - Sampling: Enable sampling randomly for testing purpose
- Debug Settings -> select sample file and row limit, select file path -> save. Under Source Options tab, After completion, I can choose no action or delete the source file after completion. Under Projection tab, I can manually edit the data type or click the Detect data type. Under Optimize tab, use the current for now. View inspect for all data types. Under Data preview, refresh to view all data. 

![image](https://user-images.githubusercontent.com/110323703/220839185-ba3be12e-5a72-4124-9bcc-a73494bf6964.png)
- Filter Transformation: Need to filter continent into only Europe (#1)-> click on the plus for a list of transformations -> Filter -> Open Expression builder -> input expression

![image](https://user-images.githubusercontent.com/110323703/220840564-a660d67b-b2f4-42f2-8458-315cc00baec8.png)
![image](https://user-images.githubusercontent.com/110323703/220840737-b17d557f-4624-4012-8417-fa680d1ec24f.png)
- Select Transformation: Need to delete column continent (#1), change column name reported_date (#4), and delete column rate_14_day (#5) -> plus, Select transformation -> move down to the mapping, for #1 and #5, need to delete the continent and rate_14_day; for #4, rename in Name as column.

![image](https://user-images.githubusercontent.com/110323703/220845266-4792e57f-1ac3-44b0-b7f2-574dc2866ec2.png)
- Pivot Transformation: Need to pivot to find out how many confirmed covid cases (cases_count) and how many covid death cases (deaths_count) per reported_day. Pivot key is the column indicator, the column being pivoted is the daily_count, and the rest of columns would form part of the group by column. Aggregate function SUM will be used to add daily_count.

![image](https://user-images.githubusercontent.com/110323703/220847370-d1b2693c-c6ba-4bf7-a567-9866ae7befd3.png)
![image](https://user-images.githubusercontent.com/110323703/220848034-5b7f3d2c-87db-4e71-bb76-cfdbb6bfae0a.png)
![image](https://user-images.githubusercontent.com/110323703/220849608-58fb86cd-98b6-45a5-821c-17fea9249ff9.png)
![image](https://user-images.githubusercontent.com/110323703/220849931-d049e54a-a83f-4617-a2a8-4e8f71adf482.png)
- Lookup Transformation: Need to create a new dataset under Datasets for the lookup file. New datasets -> ADLS 2 -> Delimited -> Browse file path -> ok -> Add Source, select the lookup file -> plus, Lookup transformation. This is acting like a left join (PivotCounts LEFT JOIN CountryLookup) -> need a select transformation right after to delete duplicate columns and rearrange columns 

![image](https://user-images.githubusercontent.com/110323703/220855071-5eeef9eb-3669-4502-8cb7-a0a1611ceaa6.png)
- Sink Transformation: It's an Output. Create a new blob container in ADLS Gen2 in Storage Explorer name "processed". Back to ADF, under Sink tab, create new dataset -> ADLS 2, delimited -> select none schema. PUBLISH ALL.

![image](https://user-images.githubusercontent.com/110323703/220857219-b56c0ae3-9c99-49c9-8fb7-11cec77ee02d.png)
- Create a new ADF pipeline that contains the newly created stream so I can execute the data flow. After finish, turn off the Data flow debug. PUBLISH. Trigger now -> go to Monitor to view it.

![image](https://user-images.githubusercontent.com/110323703/220858906-0b43ff6e-260a-4a7b-91f6-6e41513939bc.png)
![image](https://user-images.githubusercontent.com/110323703/220858630-efc3f474-81c4-4e66-a7fb-593f98887289.png)
![image](https://user-images.githubusercontent.com/110323703/220860535-24966d35-19e4-409d-aa36-1c6326a4090c.png)

- Transform hospital_admission dataset: : Go to Azure Storage Explorer -> datalake -> open the new file hospital_admissions.csv in excel -> I need to decide how I want to transform the original data depending on the need of reporting. Below is how the original file looks like.

![image](https://user-images.githubusercontent.com/110323703/227808653-8ea0c033-a6a2-4d73-8e9d-2bd9b1eaa959.png)
- Notes of transformations that I want to make:
Change #1: Drop URL column, rename date to report_date, rename year_week to reported_year_week
Change #2: Split into 2 different files: one is for Daily and one is for Weekly base on the indicator column
Change #3: In Daily file, create new columns for Daily Hospital Occupancy and Daily ICU Occupancy. In Weekly file, create new columns for Weekly Hospital Occupancy and Weekly ICU Occupancy.
Change #4: Lookup to another table dim_date to get the date info for year_week column
Change #5: Make standard for column country_code into 2 digits across all of the files (For the country_code that has 3 digits, use an external file with lookup function to lookup for its corresponding 2 digits). Keep the country_code 3 digits in there. File name country_lookup.csv. Remove continent
Change #6: Use Indicator column and Value column to derive two columns hospital_occupancy_count and icu_occupancy_count.

ALL Tranformations will be performed based on the list of changes above:
- Create Source Transformation

![image](https://user-images.githubusercontent.com/110323703/229266846-3e51a393-685a-4a48-82f0-b3bba62cec2e.png)
- Use Select Transformation to remove URL, rename date to report_date, rename year_week to reported_year_week (#1)

![image](https://user-images.githubusercontent.com/110323703/229266905-4c0583f2-38c6-41e5-8b7e-584a8b83a0f4.png)
- Use Lookup Transformation to look up on country dimension file to get 2 digits and 3 digits code and the population of the country (#5). Create another Source Transformation for the lookup file.

![image](https://user-images.githubusercontent.com/110323703/229267026-f20b5967-60b8-41a1-8980-9d612019453b.png)
- Add Select Transformation to remove continent (#5)
- Now I will split the data into 2 separate streams: one for Daily and one for Weekly (#2) indicator is the condition to split. Use Conditional Split Transformation: 

![image](https://user-images.githubusercontent.com/110323703/229267341-a5dec3f6-5276-4870-9cd4-789c956f761c.png)
![image](https://user-images.githubusercontent.com/110323703/229267319-175f5272-4514-435f-8fef-52a61b388b37.png)
- Create another Source Transformation for the dim_date lookup file. Given the dim_date table (#3, #4). Then use Derived Column Transformation to create a new column ecdc_year_week which has a required format YYYY-W##. Then to add the week_start_date and week_end_date into Weekly file, I need to create an Aggregate Transformation

![image](https://user-images.githubusercontent.com/110323703/229267897-6c6fd08d-1336-4420-a077-9b2ea9505469.png)
![image](https://user-images.githubusercontent.com/110323703/229268133-cf4b1833-2c29-42a0-8a38-b3687b05ccd3.png)
![image](https://user-images.githubusercontent.com/110323703/229268181-9ec68f97-5a34-4bd9-96d2-0824ca2a2bea.png)
- I need to join the DimDate data into the main stream. Use Join Transformation: 

![image](https://user-images.githubusercontent.com/110323703/229268512-34220c5f-3997-4c49-9ac6-468a33a61d4f.png)
![image](https://user-images.githubusercontent.com/110323703/229268614-2bc6b053-f266-41e7-a9e5-a700a73e3ed7.png)
- Use Pivot Transformation to on Daily and Weekly to get the hospital_occupancy_count/icu_occupancy_count from the indicator and the value field (#6)
    
![image](https://user-images.githubusercontent.com/110323703/229269141-a4acc86f-dc90-4e1a-8fd8-9a38f6e2d6ad.png)
![image](https://user-images.githubusercontent.com/110323703/229269232-af1fa3e1-1136-40e3-883c-c027c14841eb.png)
![image](https://user-images.githubusercontent.com/110323703/229269292-bb6fc66d-bf44-444d-837e-07f49d9bdc9a.png)
![image](https://user-images.githubusercontent.com/110323703/229269443-b310210f-2055-4f0d-9031-624feb01202d.png)
![image](https://user-images.githubusercontent.com/110323703/229269457-bcc240df-d43a-43da-aea3-b13299b3c7c3.png)
![image](https://user-images.githubusercontent.com/110323703/229269500-7b765474-dff4-463b-8d70-82b04ccd783f.png)
![image](https://user-images.githubusercontent.com/110323703/229269524-c7d93839-305d-4c49-a61a-084e7c96c767.png)
- Use Sort Tranformation for both Weekly and Daily

![image](https://user-images.githubusercontent.com/110323703/229269714-054c1054-83ec-4e1e-a0dd-495a6d36d0bf.png)
![image](https://user-images.githubusercontent.com/110323703/229269747-586a3848-b536-4840-9c2d-13a9ca5c1a5a.png)
- Select and Sink Transformation: 

![image](https://user-images.githubusercontent.com/110323703/229270484-4e257732-6b10-4e94-b208-46994f4cf493.png)
![image](https://user-images.githubusercontent.com/110323703/229270603-4d006dde-5113-4c49-aeaf-b4a62b586e8a.png)

Publish ALL. The data flow is sucessfully created. Now I need to execute the dataflow from a pipeline
- Create a pipeline -> Publish -> Trigger now. To check on the end result file, go to Azure Storage Explorer ADLS Gen2. If I want output in the same file for weekly and daily, I can go back into the Sink Transformation to make the data output into a single partition/single file and select clear the folder to remove existing files. Also update in the Sort Transformation for single partition

![image](https://user-images.githubusercontent.com/110323703/229270829-a56b1761-b0e0-45f7-9675-9bd6b40ccc7e.png)
![image](https://user-images.githubusercontent.com/110323703/229271155-ea7336d1-3354-420c-93c9-ec99a8878fb2.png)
![image](https://user-images.githubusercontent.com/110323703/229271171-f33eb064-1ec7-4845-b1df-2d0deeeca9de.png)
![image](https://user-images.githubusercontent.com/110323703/229271825-aed2b9a7-7575-435e-a2a1-de0d2dc3fd6d.png)

4. Copy Data to Azure SQL Database: after transformation, the transformed data is written back to the data lake, now I need to copy this data to Azure SQL database so it can be used for reporting.

Copy Activity - Data Lake to SQL for cases_and_deaths data
- Create SQL tables: Azure Portal -> Dashboard -> select the already created SQL database -> query editor -> log in britney/BDang1991 -> add SQL script that contains create table statements for 2 datasets.

![image](https://user-images.githubusercontent.com/110323703/229334346-98093f8e-3afa-46fc-b58e-f4c9f70b7160.png)
- Create pipeline: ADF -> new pipeline name pl_sql_cases_and_deaths_data -> select Source dataset: ds_processed_cases_and_deaths -> Sink: New, Azure SQL Database, New Linked Services, Sink dataset newly created name ds_sqlDB_cases_and_deaths
![image](https://user-images.githubusercontent.com/110323703/229334772-ea2d6ec0-a5a3-4462-af67-65fb60b1db46.png)
- Click on COPY activity, edit Source and Sink. In Sink, pre-copy script type in TRUNCATE TABLE [table name] (because I dont want duplicate data, so I need to empty the old data before new data load into) -> DEBUG to run the pipeline -> after pipeline succeeded, go into SQL editor in Azure Portal to check on the new table result.

![image](https://user-images.githubusercontent.com/110323703/231322231-7d51af6d-20d8-45e3-af07-811ea8be4df3.png)

Copy Activity - Data Lake to SQL hospital_admissions data: do similar steps as above.


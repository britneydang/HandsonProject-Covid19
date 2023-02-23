# Hands-on Project 1: Covid-19
Description: This project is about creating everything from scratch using all technologies/tools that are availalbe in Microsoft Azure Data Factory.

Data Resources:
Use COVID-19 data from the EU Center for Disease Prevention and Control website (ECDC).
- ECDC Website for Covid-19 Data - https://www.ecdc.europa.eu/en/covid-19/data
- Euro Stat Website for Population Data - https://ec.europa.eu/eurostat/estat-navtree-portlet-prod/BulkDownloadListing?file=data/tps00010.tsv.gz

Solution Architect:
- I will use the HTTP Connector within the ADF to get the COVID-19 data (data set #1) and (as a second connector) keep the Population data (data set #2) in an Azure storage account and ingest from there.
- Both data sets will be ingested into the Azure Data Lake Storage Gen2.
- I will use ADF to transform data. I will use a few different tools for the transformation: Data Flows (transformation tool) within Data Factory. For HDInsight, Azure DataBricks, Data Factory will be used as orchestration tool.
- All transformed data will be moved to Azure Data Lake Storage Gen2 for Machine Learning use.
- I also push the subset of the required data to the SQL database so it can be used for reporting.
- Visualization report such as PowerBI can be built from this data.

Technologies:
- Use Azure Data Factory for all data integration and orchestration.
- Transformation tools: Data Flows (simple transformation, code-free), HDInsight (complex, code-required, Hive/Pig), DataBricks(complex, code-required, Spark/Python).
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

3. Data Transformation - Data Flows
After ingesting data from various sources (ECDC data from website and population data from the Blob Storage) into the Data Lake, now I need to transform the data by using Data Flow (in ADF). There are many types of transformation that can be created with a data flow: source transformation, filter transformation, select transformation, pivot transformation, lookup transformation, sink transformation.
* Transform cases_deaths * dataset: Go to Azure Storage Explorer -> datalake -> open the new file cases_deaths.cvs in excel -> I need to decide how I want to transform the original data depending on the need of reporting. Below are the original file. 

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
- Filter Transformation: Need to filter continent into only Europe (#1)
- Select Transformation: Need to delete column continent (#1), change column name reported_date (#4), and delete column rate_14_day (#5)
- Pivot Transformation: Need to pivot to find out how many confirmed covid cases (cases_count) and how many covid death cases (deaths_count) per reported_day. Pivot key is the column indicator, the column being pivoted is the daily_count, and the rest of columns would form part of the group by column. Aggregate function SUM will be used to add daily_count.
- Lookup Transformation: Need to create a new dataset under Datasets for the lookup file. Drag the country_lookup into Add Source
- Sink Transformation: It's an Output. Create a new blob container in ADLS Gen2 in Storage Explorer name "processed". Settings tab -> Output to a single file -> trigger now

    
    



  



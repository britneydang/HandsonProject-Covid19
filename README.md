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
    - Delte the source file on successful copy (acing like a move, but ADF doesnt have MOVE activity): Add a DELETE activity after the COPY activity. Under properties, select original dataset, unable logging (enable if I need to keep track a list of deleted files), connect them.
![image](https://user-images.githubusercontent.com/110323703/207461737-7b1621c0-b903-4f83-9e6d-4dff23ae3302.png)



# Hands-on Project: Covid-19

Data Resources:
In this project, I will use the COVID-19 data from the EU Center for Disease Prevention and Control website (ECDC).
- ECDC Website for Covid-19 Data - https://www.ecdc.europa.eu/en/covid-19/data
- Euro Stat Website for Population Data - https://ec.europa.eu/eurostat/estat-navtree-portlet-prod/BulkDownloadListing?file=data/tps00010.tsv.gz

Solution Architect:
- I will use the HTTP Connector within the ADF to get the COVID-19 data (data set 1) and (as a second connector) keep the Population data (data set 2) in an Azure storage account and ingest from there.
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
- Create an Azure account and log in https://portal.azure.com/#home.
- Create Azure Data Factory resource via Azure portal: Create a resource -> Marketplace -> Integration -> DF. After create DF and deployment is completed, Go to Resource. Click on Launch studio.
![image](https://user-images.githubusercontent.com/110323703/207212129-9b1dd4a4-38dd-44d3-8a90-3c63199c5ea8.png)
https://portal.azure.com/#@britneydang111gmail.onmicrosoft.com/resource/subscriptions/111234fb-c6b9-407a-835e-3aa626721019/resourceGroups/COVID19-data-BritneyD/providers/Microsoft.DataFactory/factories/COVID19-reporting-ADF-BritneyD/overview

- Create a Storage Account: Create a resource -> search Storage Account. When deployment is completed, Go to Resource
![image](https://user-images.githubusercontent.com/110323703/207213881-d462b69f-e6da-41ec-beee-9aadd327cae5.png)
https://portal.azure.com/#@britneydang111gmail.onmicrosoft.com/resource/subscriptions/111234fb-c6b9-407a-835e-3aa626721019/resourcegroups/COVID19-data-BritneyD/providers/Microsoft.Storage/storageAccounts/covid19reportingbritneyd/overview

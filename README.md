# Hands-on Project: Covid-19

Data Resources:
In this project, I will use the COVID-19 data from the EU Center for Disease Prevention and Control website (ECDC).

Solution Architect:
- I will use the HTTP Connector within the ADF to get the COVID-19 data (data set 1) and (as a second connector) keep the Population data (data set 2) in an Azure storage account and ingest from there.
- Both data sets will be ingested into the Azure Data Lake Storage Gen2.
- I will use ADF to transform data. I will use a few different tools for the transformation: Data Flows (transformation tool) within Data Factory. For HDInsight, Azure DataBricks, Data Factory will be used as orchestration tool.
- All transformed data will be moved to Azure Data Lake Storage Gen2 for Machine Learning use.
- I also push the subset of the required data to the SQL database so it can be used for reporting.
- Visualization report such as PowerBI can be built from this data.

Technologies:
- Use ADF for all data integration and orchestration.
- Transformation tools: Data Flows (simple transformation, code-free), HDInsight (complex, code-required, Hive/Pig), DataBricks(complex, code-required, Spark/Python).
- Storage solutions: Azure Blob Storage, Azure Data Lake Storage Gen2 (can add the big data solutions like Hadoop, Synapse Analytics), Azure SQL Database for data warehouse
- Reporting tool: PowerBI

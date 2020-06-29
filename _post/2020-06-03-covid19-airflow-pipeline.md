---
defaults:
   # _posts
   - scope:
        path: ""
        type: posts
     values:
        layout: single
        author_profile: true
        read_time: true
        comments: true
        share: true
        related: true
---

# airflow-dag-covid19
Basic data pipeline to handle covid19 data sources utilizing Python and Airlflow.

## Setup Steps

1 pip3 install airflow

2 git clone repository on ~/airflow/dags

3 pip install -r requirements.txt

## ETL Design

After defining our data sources and goals, we start the ETL (extract, Transform, Load) design.

To summarize, an ETL process usually consist of three layers:

### Extraction layer:

Takes care of getting data out of the sources, creating a staging area that’s basically mirrors the data on source, but it enables data transformations to be more manageable and predictable.

We’ll be staging extraction data on .csv and pickle, we could use and SQL repository or any other data storage, but this two suits very well for the reality of our processing and data sources.

Pickle its an alternative way to store objects states on python, sometimes might be a better option than .csv for example, in this projection we are staging on both formats for sake of exemplification.

### Transformation layer:

Commonly summarizes the data, adds business rules and dimensions information to the model, aiming mainly on enhancing querying and reporting, providing data with quality and availability.


### Load layer:

Deliveries data to an application, reporting area, … It’s the final step where the data has to be ready for consumption.

## Credentials

To achieve our goal, we’ll need credentials to make possible for python to handle data extraction from Google Big Query (GBQ) and later on our data flow we’ll have to write data on S3, so an AWS

### Google Big Query

Googe has a very comprehensive documentation on how get this going (https://cloud.google.com/bigquery/docs/bigquery-storage-python-pandas).

For our project the objective is to download the credential.json and store it on a folder for adding it to a Variable on Airflow (explained on **Setup Airflow**).

### AWS S3

To utilize S3, a credential is needed as well. The module utilized on python is boto3, and “Real Python” has a great tutorial how to setup this credential, cheers to these guys !! (https://realpython.com/python-boto3-aws-s3/).

There’s a python file on the project (credentials.py) that deals with these resources whenever we need them.

## Data Extraction

So we start to dirt our hands finally, have in mind that we’ll be utilizing Airflow, so our project is designed to meet some structural requirements. Our tasks will run PythonOperators, so we need to provide a python callable (usually a function).
source: researchviews.blogsport.com

Besides that, google designed its interface module python/Big Query to work with pandas’ dataframes, which is really good because pandas module its a very convenient choice for data manipulation , so keep in mind that most functionalities we’ll be using further are presented on pandas, in case you missed it, check it out some examples on how to work with pandas.

Moving on, we initialize our first task on our DAG file (**covid19_datalake.py**) and looks like this :

```python
# inside DAG definition

# ... task x

extract_data = PythonOperator(
        task_id='extract_data',
	python_callable=data_extraction
)
`````
Initialize task extract_data (PythonOperator), data runs imported function data_extraction

At first glance, we notice that we have this **data_extraction** function that we’ll run whenever this task is called, and data_extraction its the function that is getting data from the sources we listed previously and creating extraction files.

On file extract_data.py we get Big Query clients needed as well initialize our DataHandler object, which does all the heavy lifting on data related manipulation.

 ```python
 def data_extraction():
     bqclient = get_bqclient()
     bqstorageclient = get_bigstorageclient()

     #Extracting raw data section
     for source in DataHandler.data_source_list:
     	data_handler = DataHandler(source)
	if data_handler.source_type == 'Big Query'
		data_handler.extract_data(bqclient, bqstorageclient)
	elif data_handler.source_type == 'CSV':
		data_handler.extract_data()
 `````
On data extraction, **DataHandler** runs through our list of 3 sources, retrieves the data and store it on two directories:

> /EXT stores .csv/.json (raw data)

> /EXT_PICKLE stores .pkl (raw data)

No data manipulation is done on this layer.

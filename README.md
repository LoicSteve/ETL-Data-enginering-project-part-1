# ETL-Data-enginering-project-part-1
Build and ETL pipelines ton serve for data analysis downstream

##  Introduction
deftunes is a new company in the music industry, offering a subscription-based app for streaming songs. Recently, they have expanded their services to include digital song purchases. With this new retail feature, deftunes requires a data pipeline to extract purchase data from their new API and operational database, enrich and model this data, and ultimately deliver the comprehensive data model for the Data Analysis team to review and gain insights. The task is to develop this pipeline, ensuring the data is accurately processed and ready for in-depth analysis
Here is the diagram with the main requirements for this project:

<img width="990" alt="Screenshot 2024-10-18 at 13 27 54" src="https://github.com/user-attachments/assets/f93814f4-0932-47d5-9477-0b16ca6bdcc3">


1. The pipeline has to follow a medallion architecture with a landing, transform and serving zone.
2. The data generated in the pipeline will be stored in the company's data lake, in this case, an S3 bucket.
3. The silver layer should use Iceberg tables, and the gold layer should be inside Redshift.
4. The pipeline should be reproducible, we will have to implement it using Terraform.
5. The data should be modelled into a star schema in the serving layer, we should use dbt for the modelling part.


Go through the jupyter notebook and the terraform folder in order to follow the different steps....

## 2 - Data Sources

The first data source you will be using is the DeFtunes operational database, which is running in RDS with a Postgres engine. This database contains a table with all the relevant information for the available songs that you can purchase. Let's connect to the table using the `%sql` magic. 

2.1. To define the connection string, go to CloudFormation Outputs in the AWS console. You will see the key `PostgresEndpoint`, copy the corresponding **Value** and replace with it the placeholder `<POSTGRES_ENDPOINT>` in the cell below (please, replace the whole placeholder including the brackets `<>`). Then run the cell code.

## 3 - Exploratory Data Analysis

To better understand the data sources, start analyzing the data types and values that come from each source. You can use the pandas library to perform Exploratory Data Analysis (EDA) on samples of data.

## 4 - ETL Pipeline with AWS Glue and Terraform

Now you will start creating the required resources and infrastructure for your data pipeline. Remember that you will use a medallion architecture.

The pipeline will be composed by the following steps:
- An extraction job to get the data from the PostgreSQL Database. This data will be stored in the landing zone of your Data Lake.
- An extraction job to get the data from the two API endpoints. This data will be stored in the landing zone of your Data Lake in JSON format.
- A transformation job that takes the raw data extracted from the PostgreSQL Database, casts some fields to the correct data types, adds some metadata and stores the dataset in Iceberg format.
- A transformation that takes the JSON data extracted from the API endpoints, normalizes some nested fields, adds metadata and stores the dataset in Iceberg format.
- The creation of some schemas in your Data Warehouse hosted in Redshift.


### 4.1 - Landing Zone

For the landing zone, you are going to create three Glue Jobs: one to extract the data from the PostgreSQL database and two to get the data from each API's endpoint. You are going to create those jobs using Terraform to guarantee that the infrastructure for each job will be always the same and changes can be tracked easily. Let's start by creating the jobs and then creating the infrastructure

4.1.1. Go to the `terraform/assets/extract_jobs` folder. You will find two scripts

- `de-c4w4a1-extract-songs-job.py`: this script extracts data from the PostgreSQL source database.
- `de-c4w4a1-api-extract-job.py`: this script extracts data from the API. Endpoints can be provided through parameters.


4.1.2. `Extract_job`. Given that you already created the scripts for the Glue Jobs, let's start by uploading them to an S3 bucket. Open the `terraform/modules/extract_job/s3.tf` file. You are already provided with some resources such as the scripts bucket and its permissions. 

4.1.3. Open the `terraform/modules/extract_job/glue.tf` file. In this file you will set all the required resources to create the Glue Jobs. 

4.1.4. Explore the rest of the files of the `extract_job` module to understand the whole infrastructure. Avoid performing further changes to other files. Here is the summary of what you can find in those files:
- In `terraform/modules/extract_job/iam.tf` file you can find the creation of the role used to execute the Glue Jobs. There is also the attachment of a policy holding the permissions. Those permissions can be found directly in the `terraform/modules/extract_job/policies.tf` file.
- The `terraform/modules/extract_job/network.tf` has the definition of the private subnet and the source database security group used to create the Glue Connection used to allow the Glue Jobs to connect to the source PostgreSQL database.
- The `terraform/modules/extract_job/variables.tf` contains the necessary input parameters for this module, while the `terraform/modules/extract_job/outputs.tf` sets the possible outputs that terraform will show in the console from this module.



# Variables and Config
export REGION=us-central1
export ZONE=us-central1-f
export PROJECT='sam-gcp-learn'
export COMPOSER_ENVIRONMENT='sam-environment'
gcloud config set compute/zone $ZONE
gcloud config set project $PROJECT

# Enable Cloud Composer API

# Create Composer Environment
gcloud composer environments create $COMPOSER_ENVIRONMENT \
    --location us-central1 \
    --project $PROJECT
    
# create a quickstart DAG


# import the DAG into composer environment
gcloud composer environments storage dags import \
  --environment $COMPOSER_ENVIRONMENT \
  --location us-central1 \
  --source quickstart.py

gcloud composer environments storage dags import \
  --environment $COMPOSER_ENVIRONMENT \
  --location us-central1 \
  --source GcsToBigQueryTriggered.py

# Create BQ dataset and table

dataset: test
table: usa_names

|Column Name | Column Type|
|:-----------|:-----------|
|state	     |STRING      |
|gender	     |STRING      |
|year	     |STRING      |
|name	     |STRING      |
|number	     |STRING      |
|created_date|STRING      |
|filename	 |STRING      |
|load_dt	 |DATE        |

# create a bucket for input files
gsutil mb gs://sam-gcp-input
gsutil mb gs://sam-gcp-output
gsutil mb gs://sam-gcp-temp

# set airflow variables
gcloud composer environments run $COMPOSER_ENVIRONMENT variables --location us-central1 \
        -- --set gcp_project $PROJECT
gcloud composer environments run $COMPOSER_ENVIRONMENT variables --location us-central1 \
        -- --set gcp_temp_location gs://sam-gcp-temp/tmp
gcloud composer environments run $COMPOSER_ENVIRONMENT variables --location us-central1 \
        -- --set gcs_completion_bucket sam-gcp-output
gcloud composer environments run $COMPOSER_ENVIRONMENT variables --location us-central1 \
        -- --set input_field_names state,gender,year,name,number,created_date
gcloud composer environments run $COMPOSER_ENVIRONMENT variables --location us-central1 \
        -- --set bq_output_table test.usa_names
gcloud composer environments run $COMPOSER_ENVIRONMENT variables --location us-central1 \
        -- --set email some-email@mycompany.com

# upload dataflow code to dags folder
create dataflow folder inside dags folder and upload the file

# creata a cloud function
https://cloud.google.com/composer/docs/how-to/using/triggering-with-gcf
- Ensure that the **DAG_NAME** property is set to _**GcsToBigQueryTriggered**_ i.e. The DAG name defined in [simple_load_dag
.py](simple_load_dag.py).

# Granting blob signing permissions to the Cloud Functions Service Account
gcloud iam service-accounts add-iam-policy-binding \
    $PROJECT@appspot.gserviceaccount.com \
    --member=serviceAccount:$PROJECT@appspot.gserviceaccount.com \
    --role=roles/iam.serviceAccountTokenCreator

# deploy gcf
gcloud functions deploy trigger_dag --runtime python37 --trigger-bucket sam-gcp-input

# place the file in input bucket to trigger the cloud fucntion 



## References: Google Cloud Documentation

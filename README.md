# serverless-neg-cf

Repository containing a first approach on the usage of [Serverless NEGs](https://cloud.google.com/load-balancing/docs/negs/serverless-neg-concepts) in conjunction with Cloud Functions.

This combination will allow us to make our functions globally available, serving content from the region closer to the user and all of them behind the same endpoint that the Load Balancer will be providing.


## 1. Activate Cloud Functions API

This will be our main product so before anything else we need to have its API enabled.

`gcloud services enable cloudfunctions.googleapis.com`

## 2. Activate Cloud Build API

Cloud Build is used behind the scenes to build the Functions that we will be deploying, therefore its API is needed. It will come in handy as well as it makes easy and accessible to debug deployment issues in Cloud Functions.

`gcloud services enable cloudbuild.googleapis.com`

After running these two commands, we can verify that our steps are correct by listing all the enabled services:

`gcloud services list`

And we can expect an output similar to this(Note that there may be other APIs enabled if you are using other products):
```
gcloud services list
NAME                              TITLE
bigquery.googleapis.com           BigQuery API
bigquerystorage.googleapis.com    BigQuery Storage API
cloudapis.googleapis.com          Google Cloud APIs
cloudbuild.googleapis.com         Cloud Build API
clouddebugger.googleapis.com      Cloud Debugger API
cloudfunctions.googleapis.com     Cloud Functions API
cloudtrace.googleapis.com         Cloud Trace API
containerregistry.googleapis.com  Container Registry API
datastore.googleapis.com          Cloud Datastore API
logging.googleapis.com            Cloud Logging API
monitoring.googleapis.com         Cloud Monitoring API
pubsub.googleapis.com             Cloud Pub/Sub API
servicemanagement.googleapis.com  Service Management API
serviceusage.googleapis.com       Service Usage API
source.googleapis.com             Legacy Cloud Source Repositories API
sql-component.googleapis.com      Cloud SQL
storage-api.googleapis.com        Google Cloud Storage JSON API
storage-component.googleapis.com  Cloud Storage
storage.googleapis.com            Cloud Storage API
```

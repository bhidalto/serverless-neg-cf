# serverless-neg-cf

Repository containing a first approach on the usage of [Serverless NEGs](https://cloud.google.com/load-balancing/docs/negs/serverless-neg-concepts) in conjunction with Cloud Functions.

This combination will allow us to make our functions run below the Load Balancer which will be providing an IP/custom domain to call our functions and also leverage the workload by distributing it in the functions attached to the NEG.


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
## 3. Deploying to Cloud Functions

### 3.1 Europe-west1 region Function1

Let's create a folder to host our first function:

```
mkdir function-eu-region-1
cd function-eu-region-1
```

Now based on the samples provided in the public [documentation](https://cloud.google.com/functions/docs/first-python#creating_a_function), let's create a sample helloworld function. Create a `main.py` file and add the following:

```
from flask import escape

def hello_http(request):
    """HTTP Cloud Function.
    Args:
        request (flask.Request): The request object.
        <https://flask.palletsprojects.com/en/1.1.x/api/#incoming-request-data>
    Returns:
        The response text, or any set of values that can be turned into a
        Response object using `make_response`
        <https://flask.palletsprojects.com/en/1.1.x/api/#flask.make_response>.
    """
    request_json = request.get_json(silent=True)
    request_args = request.args

    if request_json and 'name' in request_json:
        name = request_json['name']
    elif request_args and 'name' in request_args:
        name = request_args['name']
    else:
        name = 'World from Function 1'
    return 'Hello {}!'.format(escape(name))
```
We will also need a `requirements.txt` file that will specify our needed dependencies:
```
flask==1.1.2
```

Now we have all the needed before proceeding to deploy our Function. In order to do so, let's run the following command:

```
gcloud functions deploy [FUNCTION_NAME] \
--region europe-west1 \
--runtime python38 \
--trigger-http \
--allow-unauthenticated \
--entry-point hello_http
```
As we can appreciate from the flags we're using, we will be deploying it in `europe-west1` region and we'll be temporarily allowing unauthenticated calls to our functions. Later on, we will have them restricted.
It's very important to take into consideration the `--entry-point` flag, specially if the Cloud Function name does not match the defined function in our `main.py` file.

After our deployment is successful, we will be able to see a link like this one:
` https://europe-west1-[PROJECT_ID].cloudfunctions.net/[FUNCTION_NAME]`
So we can simply click on it to see a Hello World! message in our web browser or we can issue a curl command to quickly test it as well:
`curl  https://europe-west1-[PROJECT_ID].cloudfunctions.net/[FUNCTION_NAME]`

### 3.2 Europe-west1 region Function2

Now let's move to deploy our second function, located in a different region which we will be able to use under our Load Balancer. To deploy, we're going to repeat the steps above but we'll be hosting the code in a separate folder, for the sake of clarity.

Let's create a different folder first:
```
mkdir function-eu-region-2
cd function-eu-region-2
```
And in here create a `main.py` file and a `requirements.txt` file as we did in the previous step with a minor modification to our `main.py` file:
Replace the name to greet the user from Function 2 instead of Function 1 this time:
`name = 'World from Function 2'`

After that we deploy with the following command:

```
gcloud functions deploy [FUNCTION_NAME] \
--region europe-west1 \
--runtime python38 \
--trigger-http \
--allow-unauthenticated \
--entry-point hello_http
```

As before, we can test it navigating to the URL from the UI or by doing a GET request with curl.

Up to this point, we have our two desired Cloud Functions up and running in separate regions, but they are still serving from their own URL. Let's then add a Load Balancer with a Serverless NEG so we have a common entrypoint for both.

## 4. Setting up the Load Balancer

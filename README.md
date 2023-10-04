# IaC - Infrastructure as Code
It feels overly complex to be using a tool like terraform for my personal infrastructure.
Instead, this repository will host [gcloud][gcloud] commands that I used during setup. This way there is at least _some_
record of what I did.

## [email function][email-func]
_Example deploy script with [gcloud CLI][gcloud] to deploy as a Cloud Function._
```shell
PROJECT_ID="itmayziii"
REGION="us-central1"
FUNCTION_SA="app-email-func@itmayziii.iam.gserviceaccount.com"
RUN_SA="app-email-func@itmayziii.iam.gserviceaccount.com"
TRIGGER_SA="eventarc-trigger@itmayziii.iam.gserviceaccount.com"
# Pub/sub topic will need created separately.
TOPIC="send-email"
VERSION="v1_0_0"
APP=email
```

_Create pub/sub topic_
```shell
gcloud pubsub topics create send-email --labels="managed_by=manual,app=email"
```

_Deploy cloud function_
```shell
gcloud functions deploy SendEmail \
  --project="$PROJECT_ID"
  --gen2 \
  --trigger-topic="$TOPIC" \
  --runtime="go121" \
  --entry-point="SendEmail" \
  --region="$REGION" \
  --source="." \
  --ingress-settings="internal-only" \
  --no-allow-unauthenticated \
  --retry \
  --trigger-service-account="$TRIGGER_SA" \
  --run-service-account="$RUN_SA" \
  --service-account="$FUNCTION_SA" \
  --set-secrets="MG_API_KEY_MG_TOMMYMAY_DEV=mailgun-api-key-mg-tommymay-dev:latest" \
  --set-env-vars="PROJECT_ID=$PROJECT_ID,BUCKET=gs://itmayziii-email-templates" \
  --clear-labels \
  --update-labels="managed_by=manual,version=$VERSION"
```

## [email Package][email-package]

_Create cloud storage bucket for docs._
```shell
PROJECT_ID="itmayziii"; gcloud storage buckets create gs://itmayziii-email-package-docs \
  --project="$PROJECT_ID" \
  --location="us" \
  --default-storage-class="standard" \
  --uniform-bucket-level-access \
  --no-public-access-prevention
```

_Create cloud storage bucket for staging area for docs._ This staging area is used to copy objects to the live bucket
including metadata. This way we can first upload to the staging bucket, set the metadata for files, then copy the files
to the live bucket without ever missing metadata in the live bucket.
```shell
PROJECT_ID="itmayziii"; gcloud storage buckets create gs://itmayziii-email-package-docs-staging \
  --project="$PROJECT_ID" \
  --location="us" \
  --default-storage-class="standard" \
  --uniform-bucket-level-access \
  --no-public-access-prevention
```

_Set IAM permissions for docs storage bucket._
```shell
gsutil iam set itmayziii-email-package-docs-gcs-iam.json gs://itmayziii-email-package-docs
```

_Configure bucket to use index.html files and set 404 page._
```shell
gcloud storage buckets update gs://itmayziii-email-package-docs --web-main-page-suffix=index.html --web-error-page=404.html
```

```shell
gsutil -m setmeta -h "Content-Type:text/html" \
  -h "Cache-Control:no-cache" \
  "gs://itmayziii-email-package-docs-staging/**.html"
```

```shell
gsutil -m setmeta -h "Content-Type:text/javascript" \
  -h "Cache-Control: max-age=31536000" \
  "gs://itmayziii-email-package-docs-staging/**.js"
```

```shell
gsutil -m setmeta -h "Content-Type:text/css" \
  -h "Cache-Control: max-age=31536000" \
  "gs://itmayziii-email-package-docs-staging/**.css"
```

```shell
gsutil -m rsync -r -d gs://itmayziii-email-package-docs-staging gs://itmayziii-email-package-docs
```

```shell
gsutil -m rm gs://itmayziii-email-package-docs-staging/**
```

[email-package]: https://github.com/itmayziii/email
[email-func]: https://github.com/itmayziii/email_func
[gcloud]: https://cloud.google.com/sdk/gcloud

# IaC - Infrastructure as Code
It feels overly complex to be using a tool like terraform for my personal infrastructure.
Instead, this repository will host [gcloud][gcloud] commands that I used during setup. This way there is at least _some_
record of what I did.

## Dead Letter Logger

_Create dead letter pub/sub topic_
```shell
PROJECT_ID="itmayziii"; gcloud pubsub topics create dead-letter --project="$PROJECT_ID" --labels="managed_by=manual"
```

_Create SA for app to use_
```shell
PROJECT_ID="itmayziii" gcloud iam service-accounts create app-log-func --project="$PROJECT_ID" --display-name="Application log-func" --description="Dedicated application service account for log-func Cloud Function"
```

## [email function][email-func]

_Create pub/sub topic_
```shell
PROJECT_ID="itmayziii"; gcloud pubsub topics create send-email --project="$PROJECT_ID" --labels="managed_by=manual,app=email"
```

_Create bucket to store email templates_
```shell
gcloud storage buckets create gs://itmayziii-email-templates \
  --project="$PROJECT_ID" \
  --location="us" \
  --default-storage-class="standard" \
  --uniform-bucket-level-access \
  --public-access-prevention
```

_Set labels on bucket_
```shell
gsutil label ch -l "managed_by:manual" -l "app:email" gs://itmayziii-email-templates
```

_Enable versioning for bucket_
```shell
gsutil versioning set on gs://itmayziii-email-templates
```

_Enable lifecycle rules for bucket_
```shell
gsutil lifecycle set itmayziii-email-templates-gcs-lifecycle.json gs://itmayziii-email-templates
```

_Set IAM permissions for email templates storage bucket._
```shell
gsutil lifecycle set itmayziii-email-templates-gcs-iam.json gs://itmayziii-email-templates
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

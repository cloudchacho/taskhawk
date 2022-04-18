# Backends

Taskhawk works on cloud-native backends such as AWS and GCP. Before you can publish/consume messages, you need
to provision the required infra. This may be done manually, or, preferably, using Terraform. Taskhawk provides tools to
make infra configuration easier: see [Terraform](https://www.terraform.io/) and [Terraform Google](https://github.
com/cloudchacho/terraform-google-taskhawk) for further details.

## Queue

Taskhawk utilizes [SQS](https://aws.amazon.com/sqs/) and [Pub/Sub](https://cloud.google.com/pubsub/docs/overview) for
cloud hosted queues. These systems are very highly reliable and lets you focus on code rather than infra.

## Manual provisioning

At a high level, Taskhawk requires one topic and one subscription with dead letter queue per priority per application.

### GCP

In GCP, queues are called "subscriptions".

1. Install [gcloud](https://cloud.google.com/sdk/gcloud)
1. Authenticate with gcloud:

  ```shell
$ gcloud auth application-default login
  ```
1. Configure project (and repeat as appropriate for topics / apps):

  ```shell
$ APP=<myapp>

$ gcloud config set project <GCP_PROJECT_ID>
$ gcloud pubsub topics create taskhawk-dev-myapp-dlq
$ gcloud pubsub subscriptions create taskhawk-dev-myapp-dlq --topic taskhawk-dev-myapp-dlq
$ gcloud pubsub topics create taskhawk-dev-myapp
$ gcloud pubsub subscriptions create taskhawk-dev-myapp --topic taskhawk-dev-myapp --dead-letter-topic taskhawk-dev-myapp-dlq
  ```

### AWS

1. Install [awscli](https://aws.amazon.com/cli/)
1. Authenticate with AWS:

  ```shell
$ aws configure
  ```
1. Configure project:

  ```shell
$ APP=<myapp>

$ AWS_REGION=$(aws configure get region)
$ AWS_ACCOUNT_ID=$(aws sts get-caller-identity | jq -r '.Account')
$ aws sqs create-queue --queue-name TASKHAWK-DEV-MYAPP
$ aws sqs create-queue --queue-name TASKHAWK-DEV-MYAPP-DLQ
$ aws sqs set-queue-attributes --queue-url https://$AWS_REGION.queue.amazonaws.com/$AWS_ACCOUNT_ID/TASKHAWK-DEV-MYAPP --attributes "{\"Policy\":\"{\\\"Version\\\":\\\"2012-10-17\\\",\\\"Statement\\\":[{\\\"Action\\\":[\\\"sqs:SendMessage\\\",\\\"sqs:SendMessageBatch\\\"],\\\"Effect\\\":\\\"Allow\\\",\\\"Resource\\\":\\\"arn:aws:sqs:$AWS_REGION:$AWS_ACCOUNT_ID:TASKHAWK-DEV-MYAPP\\\",\\\"Principal\\\":{\\\"Service\\\":[\\\"sns.amazonaws.com\\\"]}}]}\",\"RedrivePolicy\":\"{\\\"deadLetterTargetArn\\\":\\\"arn:aws:sqs:$AWS_REGION:$AWS_ACCOUNT_ID:TASKHAWK-DEV-MYAPP-DLQ\\\",\\\"maxReceiveCount\\\":\\\"5\\\"}\"}"
  ```

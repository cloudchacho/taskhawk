# Taskhawk

Taskhawk is an async task execution framework (à la celery) that works on AWS and GCP, while keeping
things pretty simple and straight forward. Any unbound function can be converted into a Taskhawk task.

## Getting started

To get started with Taskhawk, choose your [backend](/taskhawk/backends) and provision the infra resources. Start a
worker process and a publisher process. Finally, publish a message from the publisher process, and you should see it 
arrive in the worker process.

Taskhawk tasks may be scheduled using capabilities of the backend (AWS Cloudwatch Events / GCP Scheduler).

## Implementations

- [Python](https://github.com/cloudchacho/taskhawk-python)
- [Golang](https://github.com/cloudchacho/taskhawk-go)

---
layout: post
title:  "Deploying to Amazon AWS S3 with jenkins and docker"
date:   2019-12-04 13:00:00 +0100
categories: jenkins technical
---

Docker is nice because it allows developers to create a deterministic virtual machine for specific uses suc has compiling code. Deterministic in the sense, that the software that has to be installed can be specified down to the exact versio nnumber. Read my other post on [how to compile projects with docker]({{ site.url }}/compiling-with-docker) for more details.

When using docker as part of Continuous Integration, one might want to upload the compiled binaries to AWS S3 for example. Granted there are now github actions that can perform this task almost effortlessly, but we can also create a docker image for this purpose.

One caveat when using docker to upload to AWS S3 is, that it requires the AWS credentials in order to log in and upload files. These credentials must not be stored in the docker container itself, as anyone with access to the container would automatically also gain access to your AWS S3. Instead I for passing the AWS credentials to the docker container on runtime. This is particularly feasible on CI systems that support secrets, which nowadays should be all of them.

So the correct solution involves two steps. First store the AWS credentials as a secret in your CI setup, such as github actions, Jenkins or Travis. In this example the two secrets are called `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`. Next the CI needs to pass these secrets to the docker container. In case of AWS S3 the credentials can be  passed as environemnt variables to the container. AWS command line will look for the environment variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.

As a prerequisite for the following command, a docker container with the [AWS Command Line Interface](https://docs.aws.amazon.com/cli/). Then pass the secret credentials to docker on startup as follows:

```
docker run --rm -it -e  AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY aws ecr get-login --no-include-email --region eu-central-1
```

With the execution of the docker container as shown above, the command `aws ecr get-login` should print login information to the terminal if the login was successful.
# go-lambda-icebreaker

<p align="left">
<img width="100" height="100" src="https://github.com/dreddick-home/go-lambda-icebreaker/raw/master/images/icebreaker.jpg">
</p>

This repo is an example of deploying a simple Go application to AWS Lambda.

<p align="left">
<img src="https://img.shields.io/github/go-mod/go-version/dreddick-home/go-lambda-icebreaker">
<img src="https://img.shields.io/github/v/release/dreddick-home/go-lambda-icebreaker">
<img src="https://github.com/dreddick-home/go-lambda-icebreaker/workflows/CICD/badge.svg">
<img src="https://img.shields.io/badge/License-Apache%202.0-blue.svg">
<img src="https://goreportcard.com/badge/github.com/dreddick-home/go-lambda-icebreaker">
</p>


## Overview

The "icebreaker" is a simple Go app which selects a random user from a slack channel and messages the channel to ask them to "break the ice". The idea is to generate fun and interesting conversation to aid with team cohesion for remote workers.

The deployment uses github actions + terraform to build the infrastructure. The cron execution is managed by Cloudwatch Events.

## Install

### Creating pipeline infrastructure

Steps to create creds and state store for the pipeline

Create a bucket to store terraform state
```bash
aws s3 mb s3://terraform.myorg.ninja
```

Create an IAM user for the pipeline deployment

```bash
USERNAME=icebreaker-deploy
aws iam create-user --user-name ${USERNAME}
```

Create and attach a policy for deployment
```bash
aws iam create-policy --policy-name ${USERNAME}-policy --policy-document '{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "lambda:*",
                "events:*"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:*"
            ],
            "Resource": [
              "arn:aws:s3:::terraform.myorg.ninja/*",
              "arn:aws:s3:::terraform.myorg.ninja"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:*"
            ],
            "Resource": [
              "arn:aws:iam::*:user/icebreaker*",
              "arn:aws:iam::*:policy/icebreaker*",
              "arn:aws:iam::*:role/icebreaker*
            ]
        }
    ]
}'

POLICY_ARN=$(aws iam list-policies --scope Local --output text --query 'Policies[?PolicyName==`'${USERNAME}-policy'`].Arn')

aws iam attach-user-policy --user-name ${USERNAME} --policy-arn ${POLICY_ARN}
```

Create an access key for the pipeline user
```bash
aws iam create-access-key --user-name ${USERNAME}
```

Store the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY as Github secrets within the project

### Configuring Slack

Create an internal app:
* https://api.slack.com/apps
* Create App
* Add permissions "channels:read", "chat:write", "users:read" in the "OAuth & Permissions" section
* Copy "Bot User OAuth Access Token" - this is the token which will be used by the command
* Install app
* Invite app into channel in slack - for example: "/invite icebreaker"

Store the Bot User Oauth Token as TOKEN in Github secrets
Store the ChannelId (NOT name) in Github secrets as CHANNEL_ID


## Releases

See https://github.com/dreddick-home/go-lambda-icebreaker/releases

## TODO

* Scan message history to ensure same person doesnt get picked twice in a row
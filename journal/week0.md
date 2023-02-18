# Week 0 — Billing and Architecturegit

## Infrastructure Diagram
![Infrastructure Diagram] (/assets/BootCampLogicalIfra.png)

[To view infra diagram on lucid chart](https://lucid.app/lucidchart/4128cae8-59cf-4e58-955d-93dcf9f66230/edit?viewport_loc=-424%2C-1365%2C2994%2C1481%2C0_0&invitationId=inv_05b99143-8c8f-46b9-b980-b46f635447dd)
### Required Homework
1) create a admin user
2) Generate AWS Creadentials
3) install AWS CLI
4) Create a billing Alarm
5) Create a Budget

### 1) Install AWS CLI

- I have install the AWS CLI when my Gitpod enviroment.
- I have also installed aws CLI in my linux machine.
- Commands to install cli - https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html


`.gitpod.yml` File

```sh
tasks:
  - name: aws-cli
    env:
      AWS_CLI_AUTO_PROMPT: on-partial
    init: |
      cd /workspace
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip awscliv2.zip
      sudo ./aws/install
      rm -rf awscliv2.zip
      cd $THEIA_WORKSPACE_ROOT
```

### 2) Create a new User and Generate AWS Credentials

- In IAM console new user is created with name admin (https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1#/users)
- `Enable console access` for the user
- Create a new `Admin` Group and apply `AdministratorAccess`
- Create the user and go find and click into the user
- Click on `Security Credentials` and `Create Access Key`
- Choose AWS CLI Access
- Download the CSV with the credentials

### Set Env Vars on git POD

We will set these credentials for the current bash terminal
```
export AWS_ACCESS_KEY_ID=""
export AWS_SECRET_ACCESS_KEY=""
export AWS_DEFAULT_REGION=us-east-1
```

We'll tell Gitpod to remember these credentials if we relaunch our workspaces
```
gp env AWS_ACCESS_KEY_ID=""
gp env AWS_SECRET_ACCESS_KEY=""
gp env AWS_DEFAULT_REGION=us-east-1
```

### Check that the AWS CLI is working and you are the expected user

```sh
aws sts get-caller-identity
```

You should see something like this: (I have removed some charactors from user id and account id)
```json
{
    "UserId": "AIDA4JTM55RI3R6VBSA",
    "Account": "8452638945",
    "Arn": "arn:aws:iam::8452638945:user/admin"
}
```

## Enable Billing 

We need to turn on Billing Alerts to recieve alerts...


- In your Root Account go to the [Billing Page](https://console.aws.amazon.com/billing/)
- Under `Billing Preferences` Choose `Receive Billing Alerts`
- Save Preferences


### Create SNS Topic

- before creating alarm we req an sns topic
- The SNS topic is what will delivery us an alert when we get overbilled
- [aws sns create-topic](https://docs.aws.amazon.com/cli/latest/reference/sns/create-topic.html)

create a SNS Topic
```sh
aws sns create-topic --name billing-alarm
```
which will return a TopicARN

Now, we need to create SNS subscription and also need to accecpt from email.
```sh
aws sns subscribe \
    --topic-arn TopicARN \
    --protocol email \
    --notification-endpoint your@email.com
```

Check your email and confirm the subscription

#### Create Alarm
- refrence liks
- [aws cloudwatch put-metric-alarm](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html)
- [Create an Alarm via AWS CLI](https://aws.amazon.com/premiumsupport/knowledge-center/cloudwatch-estimatedcharges-alarm/)

- Need to update the configuration json script with the TopicARN we generated earlier
- created put matrix alarm for that updated the json file.

```sh
aws cloudwatch put-metric-alarm --cli-input-json file://aws/json/alarm_config.json
```

## Create an AWS Budget

[to create aws budget](https://docs.aws.amazon.com/cli/latest/reference/budgets/create-budget.html)

Get your AWS Account ID
```sh
aws sts get-caller-identity --query Account --output text
```

- Supply your AWS Account ID
- Update the json files
- execute below commands

```sh
aws budgets create-budget \
    --account-id AccountID \
    --budget file://aws/json/budget.json \
    --notifications-with-subscribers file://aws/json/budget-notification-subs.json
```
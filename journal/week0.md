# Prerequisites
I have managed to complete the prerequisites of the bootcamp, as described by Andrew and the team into the [FREE AWS Cloud Project Bootcamp](https://www.youtube.com/watch?v=8b8SvQHc4Pk&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv) Playlist, the requirements was as below:
- Create a DVCS account(GitHub Account): I already have one.
- Create CDE account: I have created Gitpod Account, and i'm planing to activate Github Codespaces Account once reaching the Gitpod free tier limit.
- Create an AWS account: Successfully created with the free tier.
- Custom Domain Name: One consideration that should be taken into account as highlighted in the outline document is to ensure that the domain provider allows you to update the nameservers, so we can point the domain to AWS Namespaces.
- Diagramming application: Created a free Lucidchart account, so we can drow our Conceptual, Logical and physical diagrams.
- Distributed tracing tool: Created an HoneyComb.io account.
- Bug tracking tool: Created an Rollbar account.

# Week 0 — Billing and Architecture

## Enable the needed privileges for Gitpod:
By default when sign up into Gitpod using your github account it'll only be granted with `user:email` permission, we need an extra permission so we can push new commits into Github, in our case since it's public repo we'll add the `public_repo` permission to have the write permissions:
- Under the user logo in the top right -> `User Settings`.
- Go to `Integrations`.
- In Github click into `Edit Permissions` -> check `public_repo` -> then authorize the gitpod to have `public_repo` permission.
![Gitpod public_repo permission](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week0/00-Gitpod_public_repo_permission.png)

## Getting the AWS CLI Working
A part of any AWS cloud project, AWS CLI is needed to compelte administration task on our AWS cloud.

### Install AWS CLI
- Add a task into `.gitpod.yml` file in the Project's root folder, to automate the installion of AWS CLI.
- The task sould enable the partial autoprompt mode, so we can have suggesion if we intend to, by using the <Tab> key.
- The instalation guide: [AWS CLI Install Instructions](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)(Linux)

Append the below code at the top of `.gitpod.yml` file, to include the desired task (as explained above):
```
tasks:
  - name: aws-cli
    env:
      AWS_CLI_AUTO_PROMPT: on-partial
    init: |
      cd /workspace
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip awscliv2.zip
      sudo ./aws/install
      cd $THEIA_WORKSPACE_ROOT
```

### Create a new User and Generate AWS Credentials
After setting up our AWS account with a free tier, we'll be login with the Root account, it's recommanded to work with an IAM user instead of root account and enabling MFA authentication for both Root and IAM users.
- Create a new IAM group with "AdministratorsAccess" previleges:
![IAM Admin Group](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week0/01-New_IAM_Admin_Group.png)
- Create a new IAM user and and assign it to the IAM group created in the previous step:
![IAM User](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week0/02-New_IAM_User.png)
- Under `Security Credentials`, create `Access Key` for the IAM user with AWS CLI Access:
![IAM User](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week0/03-Access_Key.png)
- Make sure to save the CSV credentials file into a safe place.

### Set Env Vars
- Now let's go to our Gitpod workspace and set `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` and `AWS_DEFAULT_REGION` as environment variables:
```
$ export AWS_ACCESS_KEY_ID="<REDACTED>"
$ export AWS_SECRET_ACCESS_KEY="<REDACTED>"
$ export AWS_DEFAULT_REGION="<REDACTED>"
```

- For future use we have saved those variables into Gitpod environment variables:
```
$ gp env AWS_ACCESS_KEY_ID="<REDACTED>"
$ gp env AWS_SECRET_ACCESS_KEY="<REDACTED>"
$ gp env AWS_DEFAULT_REGION="<REDACTED>"
```

- Confirming that Gitpod has succufully saved those variables:
![Gitpod environment variables](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week0/04-Gitpod_env_variables.png)

### Check that the AWS CLI is working and you are the expected user
- Make sure to go into a region with CloudShell access (can be checked with the shell icon in the top bar), and lanch the CloudShell to check our identity:
![Check the Account in AWS CloudShell](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week0/05-AWS_CloudShell.png)

## Billing Alerts

### Enable Billing:
First we need to enable belling Alert:
- Go to the [Billing Page](https://console.aws.amazon.com/billing/)
- Under `Billing Preferences` we checked the `Receive Billing Alerts` and add the email where to receive the billing Alerts.

## Creating a Billing Alarm

We have two way to set billing alert:
- The old method: Using CloudWatch Alarm, we have to make sure that we are in `North Virginia` region (`us-east-1`).
- The new method: Using Budget.

### CloudWatch Alarm

#### Create SNS Topic
Amazon Simple Notification Service or SNS is a managed messaging service for communication, it's service that will deliver us the alert, in our case, via email.
From [AWS Docs](https://docs.aws.amazon.com/cli/latest/reference/sns/create-topic.html), we create an SNS topic:
```
aws sns create-topic --name billing-alarm
```
It will return `TopicARN` that is used along with the email to create an email subscription as below:
```
$ aws sns subscribe \
    --topic-arn <TopicARN> \
    --protocol email \
    --notification-endpoint <your@email.com>
```
Check our email and confirm the subscription.

Check the Amazon SNS subscription:
![Amazon SNS subscription](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week0/06-AWS_SNS.png)

##### Create Alarm
Refering to [this link](https://aws.amazon.com/premiumsupport/knowledge-center/cloudwatch-estimatedcharges-alarm/) to create an alarm via AWS CLI to monitor daily EstimatedCharges and trigger a CloudWatch alarm based on my usage threshold.
The [AWS documentation of cloudwatch put-metric-alarm](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html).

- Update json config file using our `TopicARN`.
```
$ aws cloudwatch put-metric-alarm --cli-input-json file://aws/json/alarm-config.json
```
![Billing Alarms](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week0/07-Billing_Alarms.png)

### Create an AWS Budget
Refering to AWS CLI Command Reference [Examples](https://docs.aws.amazon.com/cli/latest/reference/budgets/create-budget.html#examples).
```
$ export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
$ gp env AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
```

We are using also here the json file for the config since it's a long config.

```
$ aws budgets create-budget \
    --account-id $AWS_ACCOUNT_ID \
    --budget file://aws/json/budget.json \
    --notifications-with-subscribers file://aws/json/budget-notifications-with-subscribers.json
```
![Budgets](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week0/08-Budgets.png)

## Architecture Diagram
Flowing up the awesome discussion during the live stram, we had a lot of good pracices to properly understand the business needs before solutioning our project, one of the discussed topics:
- Iron triangle: budget, scope and schedule.
- Good architecture: meet the Requirement, adresses the Risks, Assumptions and Constrains
- After gathering the above we can then create our design

### Conceptual Diagram
A deign that has a commun language between technical team(devolopers, cloud engineers,...), marketing/financial team, investiors, CFO and CEO.
This diagram should be constracted by all the concerned parties.
Below the designed diagram:
![Conceptual Diagram](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week0/09-Cruddur_Conceptual-Diagram.png)
[Conceptual Diagram Lucidchart](https://lucid.app/lucidchart/6dc17b10-082b-47fb-95fa-9147238bec03/edit?viewport_loc=-1620%2C53%2C1480%2C673%2CmnZy554frJXa&invitationId=inv_04df7f9c-e635-4ca9-8cc5-8854c816daae)

### Logical Diagram
  ![Logical Diagram](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week0/10-Cruddur_Logical-Diagram.png)
  [Logical Diagram Lucidchart](https://lucid.app/lucidchart/8fcb5bf3-5424-43d3-a0aa-b238abb47bd8/edit?viewport_loc=-240%2C371%2C1565%2C713%2C0_0&invitationId=inv_98c30f9e-4009-4567-a2aa-c2ec65d51f8b)

### Define a workload

With the IT increase, modern systems are becoming more complex, to address the complexity we need to follow a best practices from the beginning of our project, with the help of enterprise architecture framework, we have seen during the live stream: TOGAF framework and C4 Model.
For the purpose of our project we are using AWS Well-Architected framework, AWS Well-Architected framework has six pillars: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization and Sustainability. Each pillar define a set of questions with the appropriate best practices and we have to check which one we are following and note down the others to decide if we should follow them or if they do not apply to our use-case.


## Security Consideration

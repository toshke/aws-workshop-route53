## Step 1 - Initializing..

In this step we will create a VPC with some Route53 resolvers for VPC bound DNS traffic. 

Resources will be created in this step with mix of IAC (Infrastructure as a code) and manual 
(aka ClickOps) method. We will be using IAC for complex task of setting up a whole aws cloud based
network, with manual addition of Route53 resolvers. Additionally, we will configure these resolvers
to send logs to CloudWatch. 


#### Create VPC network

As a VPC network has a number of elements 

```
cd step1 
aws cloudformation deploy --template-file ./vpc-template.yaml \
    --stack-name "meetup-vpc" \
    --region ap-southeast-2
```

VPC Template was built to work specifically with Sydney region. 
Upon successfull operation you should be greeted with following message

```
Waiting for changeset to be created..
Waiting for stack create/update to complete
Successfully created/updated stack - meetup-vpc
```

#### Enabling DNS traffic logging

... TBD


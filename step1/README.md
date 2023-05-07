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

DNS traffic within VPC can be logged to multiple 
destinations, including S3 Object storage, CloudWatch logs
and Kinesis Firehose stream. 

For the purposes of this workshop we will be using CloudWatch logs. 
On top of IoC created VPC created in the previous step we will create 
Route 53 Resolver Query logging configuration. This will effectively tell 
the AWS to record any DNS query originating from given VPCs into the 
default DNS resolvers. 

1 - [Go to query logging configuration home page](https://ap-southeast-2.console.aws.amazon.com/route53resolver/home?region=ap-southeast-2#/query-logging)

2 - Click "Configure query logging"

3 - Use following options
  - Name: `meetup-querylog-config`
  - Destination: Cloud Watch 
  - CloudWatch logs group: Select *Create log group*
  - New log group name: `/aws/route53/meetup-query-logging` 

  <img width="823" alt="Screenshot 2023-05-07 at 10 33 11 pm" src="https://user-images.githubusercontent.com/1170273/236677897-5bdde9e2-1e9f-4e06-8abe-78f1d97f9752.png">

4 - Click *Add VPC* button to select previously created VPC, 
    select **Simple VPC**, click "Add" button

5 - *(Optional)* add "purpose=aws-workshop" tag 

6 - Click *Configure query logging* button

You should see a message like one below confirming that Route53 Query Logging
configuration has been created and associated with a newly created VPC

<img width="743" alt="Screenshot 2023-05-07 at 10 37 51 pm" src="https://user-images.githubusercontent.com/1170273/236678076-4826076e-392a-44ae-bcc3-5e161c096443.png">


#### Validate the configuration

You can validate that CloudWatch log group has been created 
in [CloudWatch console](https://ap-southeast-2.console.aws.amazon.com/cloudwatch/home?region=ap-southeast-2#logsV2:log-groups/log-group/$252Faws$252Froute53$252Fmeetup-query-logging). 

At this point, there are is no user deployed nodes on the network, 
so the log group should be free of any log streams. 

[Proceed to step2 to deploy Ec2 instances in public subnet to start
seeing some log entries coming through](../step2/README.md)
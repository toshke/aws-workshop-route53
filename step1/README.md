## Step 1 - Initializing..

In this step we will create a VPC with some Route53 resolvers for VPC bound DNS traffic. 

Resources will be created in this step with mix of IAC (Infrastructure as a code) and manual 
(aka ClickOps) method. We will be using IAC for complex task of setting up a whole aws cloud based
network, with manual addition of Route53 resolvers. Additionally, we will configure these resolvers
to send logs to CloudWatch. 


#### Create VPC network

As a VPC network has a number of elements 


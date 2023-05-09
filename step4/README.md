## Step 4 - SOC integration


Challenges

- Create a lambda function to synchronise S3 object with domain lists 
- Restrict NACLs to allow only DNS traffic to default DNS resolvers within VPC
- Alert when a lookup to domain from the block lists is performed using CloudWatch alarms 
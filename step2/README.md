## Step 2 - Deploy EC2 instance, monitor DNS traffic

In this step, you will be deploying ec2 instance to a public subnet of a VPC 
deployed in previous step. While this can be performed using IAC, doing it
step by step is good opprotunity to discuss some of the security features
of the EC2 service. 

#### Create SSH key

```
ssh-keygen -f ./workshop.key
```

Alternatively you can use your existing default SSH key in `$HOME/.ssh` folder 

To register this rsa key with your AWS account execute following command 
in the bash shell (from the same folder where they key has been created)

```
aws ec2 import-key-pair --region ap-southeast-2 --key-name workshop-route53-key --public-key-material fileb://./workshop.key.pub
```

#### Deploy the instance

First, determine public subnet id of the VPC deployed in the previous step


```
vpcid=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=simple-vpc --query 'Vpcs[0].VpcId' --output text) 
subnetid=$(aws ec2 describe-subnets --filters Name=vpc-id,Values=$vpcid Name=tag:Name,Values=simple-vpc-PublicA --query 'Subnets[0].SubnetId' --output text)

echo "Instance to be launched in vpc $vpcid / subnet $subnetid"
```

We wil need to create new security group allowing our IP address to access the instance on port 22:


```
# required for ZSH users
export PROMPT_EOL_MARK=
export AWS_REGION="ap-southeast-2"
export AWS_PAGER=""
# determine your own ip

ip=$(curl ipinfo.io | jq -r '.ip')

## if you don't have curl or jq installed, you can visit https://ipinfo.io in your browser
##   in order to obtain your IP address 

# now we create the vpc security group with the values from the previous step
sgid=$( aws ec2 create-security-group --vpc-id $vpcid --group-name meetup-ssh-group --description "Allow SSH access from $ip" --query GroupId --output text)

echo "Port 22 to be granted from ip $ip on SG $sgid"
aws ec2 authorize-security-group-ingress --port 22 --protocol tcp --group-id $sgid --cidr "$ip/32"
```


With the key and the Security group rule ready, only missing piece is image id to launch the instance. 
Follow the instrunctions below for these 


```
# get the default AL2 ami 
amiid=$(aws ssm get-parameter --name '/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-6.1-x86_64'  --query 'Parameter.Value' --output text)

# run the instance
aws ec2 run-instances --image-id $amiid --subnet-id $subnetid --security-group-ids $sgid --instance-type t2.micro --key-name workshop-route53-key --query 'Instances[0]' --output table --associate-public-ip-address


## at this point you should be presented with table layout about instance information
## you can grab the instance ID from this screen, or alternatively use ec2 describe-instances 
## command (or Web console)
```



#### Monitor the traffic

Instance ID should apppear in the list of the Query logging CloudWatch streams you saw in step1. 

Jump to [CloudWatch console], navigate to "Logs / Log groups", and filter for "/aws/route53/meetup"

Once you are presented with the list of the VPC Query logging log streams, search for your instance id
using filter text box - you can inspect the log entries for your instance. 

<img width="821" alt="Screenshot 2023-05-08 at 10 00 57 pm" src="https://user-images.githubusercontent.com/1170273/236818694-f1c1a6d0-ff18-49c2-9df8-ab66fd9c1f1b.png">
List of all log streams, filtered by the instance id

<img width="1125" alt="Screenshot 2023-05-08 at 9 56 42 pm" src="https://user-images.githubusercontent.com/1170273/236818720-637341a0-74b0-47a4-9245-6ccc5a155c5f.png">
Details of CNAME lookup for a S3 bucket performed during the instance boot





#### Login to instance, generate some user traffic 

Now that instance has been spun up in IOC created VPC and Subnet, and with previously created
SSH key, given that security group has been properly configured you can connnect to it using 
ssh utility

```
# make sure to wait for 10 seconds or so before executing this command while instance gets ready 
# to accept incoming connections
instance_ip=$(aws ec2 describe-instances --filters Name=instance-state-name,Values=running Name=key-name,Values=workshop-route53-key --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)

ssh -i step2/workshop.key "ec2-user@${instance_ip}" 

#### you should be logged into the cloud instance at this point
# perfrom lookup 
nslookup meetup.com
```

#### Inspect the user generated traffic

Repeat the steps from `Monitor the traffic` section on this page, and locate teh log entries 
having `meetup.com` entries that you have just performed while ssh'ed in the instance. 


[Proceed to step3 to block unwanted traffic](../step3/README.md)
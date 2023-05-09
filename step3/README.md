## Step 3 - Block unwanted traffic

In this step we'll get opprotunity to work with Domain Lists - essentially grouping 
of domain names, and put them to work alongside some rules, that tie domain, 
vpc and action into a filtering rule.

### 1 - Create a domain list

Let's start by creating a domain list from Route53 console.
[Go to Domain lists home page](https://ap-southeast-2.console.aws.amazon.com/route53resolverdnsfirewall/home?region=ap-southeast-2#/domainlist/)

Click on *Add Domain list* -> 

- Set domain list name: `aws-meetup-workshop-dl` 

- Set following domains in "Domain list per line"

```
threat.actor
ebay.com
```

- *(Optional)* add `purpose=workshop` tag

- Click **Add domain list**

Alternatively inspect the [documentation](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/route53resolver/create-firewall-domain-list.html) for the 
`aws route53resolver create-firewall-domain-list` 

### 2 - Create a filtering rule group, and associate with the VPC

Multiple DNS traffic filtering rules are ordered into groups. 
This way a priority/order of execution can be set between different rules

First we'll create a rule group, and subsequently create a rule
to block all the domains in previously created domain list

You can perform this step from web UI, even though instrunctions below 
are for CLI usage

```
export PROMPT_EOL_MARK=
export AWS_REGION="ap-southeast-2"
export AWS_PAGER=""

# create a rule group
fgid=$(aws --region ap-southeast-2 route53resolver create-firewall-rule-group --name meetup-workshop-rg  --query 'FirewallRuleGroup.Id' --output text)

aws route53resolver list-firewall-rule-groups 

# associate rule group with a vpc
vpcid=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=simple-vpc --query 'Vpcs[0].VpcId' --output text)

echo "Associating firewall group $fgid with vpc $vpcid"

aws route53resolver associate-firewall-rule-group --firewall-rule-group-id $fgid --vpc-id $vpcid --priority "101" --name meetup-workshop-rg
```

### 3 - Test the DNS lookup from the instance itself

At this point you can test dns traffic from the instance. 
Assuming ssh keys and instance you spun in in step2 are still there


```

# get the instance id
instance_ip=$(aws ec2 describe-instances --filters Name=instance-state-name,Values=running Name=key-name,Values=workshop-route53-key --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)

# login to instance
ssh -i step2/workshop.key "ec2-user@${instance_ip}" 


# once securely logged into the instance execute nslookup

nslookup threat.actor
nslookup ebay.com
```

you should get valid DNS response at this point such as one below
```
[ec2-user@ip-10-100-0-18 ~]$ nslookup ebay.com
Server:		10.100.0.2
Address:	10.100.0.2#53

Non-authoritative answer:
Name:	ebay.com
Address: 209.140.139.232
Name:	ebay.com
Address: 209.140.136.23
Name:	ebay.com
Address: 209.140.136.254

[ec2-user@ip-10-100-0-18 ~]$ nslookup threat.actor
Server:		10.100.0.2
Address:	10.100.0.2#53

Non-authoritative answer:
Name:	threat.actor
Address: 35.209.123.34

```

At this point, you can go back to last part of step2 to see the
traffic coming through from your instance

### 4 - Associate AWS managed list in blocking mode

At this point we have create a rule group, connected it with VPC, now we only need 
a connection with the domain list. For demonstration purposes we will return empty 
response for the DNS lookups for domains on the list

```
fgid=$( aws route53resolver list-firewall-rule-groups  --query "FirewallRuleGroups[?Name=='meetup-workshop-rg'][].Id" --output text)
dlid=$(aws route53resolver list-firewall-domain-lists --query "FirewallDomainLists[?Name == 'aws-meetup-workshop-dl'][].Id" --output text)

aws route53resolver create-firewall-rule --firewall-rule-group-id $fgid --firewall-domain-list-id $dlid --action BLOCK --block-response NODATA --name block-dl-domains --priority 101

```

### Verify the traffic is blocked

Repeat the instrunctions from step 3. You should be met with following 
response 

```
[ec2-user@ip-10-100-0-18 ~]$ nslookup threat.actor
Server:		10.100.0.2
Address:	10.100.0.2#53

Non-authoritative answer:
*** Can't find threat.actor: No answer

[ec2-user@ip-10-100-0-18 ~]$ nslookup ebay.com
Server:		10.100.0.2
Address:	10.100.0.2#53

Non-authoritative answer:
*** Can't find ebay.com: No answer
```


Congratulations, you have completed all prescriptive steps 
in this workshop. 

[Go to step4 for some ideas on additional practice tasks](../step4/README.md)
# aws-workshop-route53
AWS Route53 Workshop - monitoring and filtering 


## Prereqs

- Computer with a working internet connection
- AWS Account 
- AWS CLI v2 - [install instrunctions](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- Bash or ZSH scripting environment  (Instrunctions are tested with BASH)
- `ssh` 
- Optional `jq` and `curl`

## Common mistakes

- User not in the correct AWS region

[Kick off the workshop by going to step1](step1/README.md)


## Cleanup 

To remove the resources created in this workshop

[Go to route53 dns firewall rule groups console](https://ap-southeast-2.console.aws.amazon.com/route53resolverdnsfirewall/home?region=ap-southeast-2#/rulegroup/), delete each rule in the group, 
dissaciate from VPC, and delete the group

[Go to route53 dns firewall domain lists console](https://ap-southeast-2.console.aws.amazon.com/route53resolverdnsfirewall/home?region=ap-southeast-2#/domainlist/) and remove created domain list. 

[Terminate the instance you have created from ec2 console]()

[Go to CloudFormation and delete the meetup-vpc stack](https://ap-southeast-2.console.aws.amazon.com/cloudformation/home?region=ap-southeast-2#/stacks?filteringText=&filteringStatus=active&viewNested=true)



TBD : Clean up instance and VPC
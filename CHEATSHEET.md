# CLD 2024 - Repository

## Commands glossary

### Link to documentation
[AWS documentation](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)   
[EC2 documentation](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/index.html#cli-aws-ec2)

### AWS global commands

**Creating the credentials which will be stored in the `~/.aws` folder :**
```shell
aws configure
```

**Commands coming after running `aws configure` :**
```shell
`aws configure` subcommands
<br> `AWS Access Key ID [None]:` Access key ID
<br> `AWS Secret Access Key [None]:` Secret key
<br> `Default region name [None]:` eu-west-3
<br> `Default output format [None]:` ? 
```

**Get details from the user using the CLI with credentials stored in `~/.aws/credentials` :**
```shell
aws sts get-caller-identity
```


### AWS ec2 commands

**Create a ec2 subnet :**
```shell
aws ec2 create-subnet \
--tag-specification "ResourceType=subnet,Tags=[{Key=Name,Value=<SUBNET_NAME>}]" \
--vpc-id <VPC_ID> \
--cidr-block <IP_ADDR> 
```

**Create a ec2 route table :**
```shell
aws ec2 create-route-table \
--tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=<ROUTE_TABLE_NAME>}]' \
--vpc-id <VPC_ID>
```

**Associate the route table to the subnet :**
```shell
aws ec2 associate-route-table \
--subnet-id <SUBNET_ID> \
--route-table-id <ROUTE_TABLE_ID>
```

**Create a security group :**
```shell
aws ec2 create-security-group \
--group-name <SECURITY_GROUP_NAME> \
--description <"DESCRIPTION"> \
--vpc-id <VPC_ID>
```

**Describe the subnetworks :**
```shell
aws ec2 describe-subnets \
--region <REGION>
```

**Authorize a service on the VPC (SSH, HTTP, ...) :**
```shell
aws ec2 authorize-security-group-ingress  \
--group-id <GROUPE_ID> \
--protocol <TRANSPORT_PROTOCOL> \
--port <SERVICE_PORT> \
--cidr 0.0.0.0/0
```

**Create an ec2 instance :**
```shell
aws ec2 run-instances \
--image-id <IMAGE_ID> \
--count <NB> \
--instance-type <TYPE> \
--key-name <INSTANCE_NAME> \
--private-ip-address <PRIVATE_IP> \
--security-group-ids <SECURITY_GROUP_ID> \
--subnet-id <SUBNET_ID> \
--tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=<NAME>}]"

Example:
aws ec2 run-instances \
--image-id ami-00b3a1b7cfab20134 \
--count 1 \
--instance-type t3.micro \
--key-name CLD_KEY_DRUPAL_DEVOPSTEAM09 \
--private-ip-address 10.0.9.10 \
--security-group-ids sg-0442609af9e1beac9 \
--subnet-id subnet-0ad378d360ff015d1 \
--tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=EC2_PRIVATE_DRUPAL_DEVOPSTEAM09}]"
```

### SSH commands

**Connect to a server (the default port number is 22) :**
```shell
ssh <USERNAME>@<HOST_IP_ADDR> -i <KEY_PATH> (-p <PORT>)

Example:
ssh devopsteam09@15.188.43.46 -i ~/.ssh/CLD_KEY_DMZ_SSH_CLD_DEVOPSTEAM09.pem
```
**Connect to the ec2 subnet :**
```shell
ssh devopsteam09@15.188.43.46 -i ~/.ssh/CLD_KEY_DMZ_SSH_CLD_DEVOPSTEAM09.pem -L 2223:10.0.9.10:22
```

**Connect to the ec2 subnet with SSH :**
```shell
ssh -J cld_dmz bitnami@10.0.9.10 -i ~/.ssh/CLD_KEY_DRUPAL_DEVOPSTEAM09.pem
```



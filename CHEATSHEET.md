# CLD 2024 - Repository

## Commands glossary

### Link to documentation
[AWS documentation](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)   
[EC2 documentation](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/index.html#cli-aws-ec2)

### Global commands
|Commands|Description|
|--|--|
|`aws configure`|creating the credentials which will be stored in the `~/.aws` folder.|
|`aws configure` subcommands <br> `AWS Access Key ID [None]:` Access key ID <br> `AWS Secret Access Key [None]:` Secret key <br> `Default region name [None]:` eu-west-3 <br> `Default output format [None]:` ? |commands coming after running `aws configure`|
|`aws sts get-caller-identity`|get details from the user using the CLI with credentials stored in `~/.aws/credentials`|

### ec2 commands
|Commands|Description|
|--|--|
|`aws ec2 create-subnet --tag-specification "ResourceType=subnet,Tags=[{Key=Name,Value=<SUBNET_NAME>}]" --vpc-id <VPC_ID> --cidr-block <IP_ADDR>`|create a subnet|
|`aws ec2 create-route-table --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=<ROUTE_TABLE_NAME>}] --vpc-id <VPC_ID>'`|create a route table|
|`aws ec2 associate-route-table --subnet-id <SUBNET_ID> --route-table-id <ROUTE_TABLE_ID>`|associate the route table to the subnet|
|`aws ec2 create-security-group --group-name <SECURITY_GROUP_NAME> --description <"DESCRIPTION"> --vpc-id <VPC_ID>`|create a security group|
|`aws ec2 describe-subnets --region <REGION>`|list the subnetworks|
|`aws ec2 authorize-security-group-ingress --group-id <GROUPE_ID> --protocol <TRANSPORT_PROTOCOL> --port <PORT> --cidr 0.0.0.0/0`|authorize SSH on the VPC|
|`aws ec2 authorize-security-group-ingress --group-id <GROUPE_ID> --protocol <TRANSPORT_PROTOCOL> --port <PORT> --cidr 0.0.0.0/0`|authorize HTTP on VPC|

### ec2 commands

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

### SSH commands
|Command|Description|
|--|--|
|`ssh [username]@[host_ip_address]` |connecting to a server|
|`ssh devopsteam09@15.188.43.46 -i ~/.ssh/CLD_KEY_DMZ_SSH_CLD_DEVOPSTEAM09.pem`|used command|
|`ssh [username]@[host_ip_address] -p [port]` |the default port number is 22|


### EC2 commands
|Command|Description|
|--|--|
|aws ec2 run-instances \
 --image-id ami-00b3a1b7cfab20134 \
 --count 1 \
 --instance-type t3.micro \
 --key-name CLD_KEY_DRUPAL_DEVOPSTEAM09 \
 --private-ip-address 10.0.9.10 \
 --security-group-ids sg-0442609af9e1beac9 \
 --subnet-id subnet-0ad378d360ff015d1 \
 --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=EC2_PRIVATE_DRUPAL_DEVOPSTEAM09}]"|create an ec2 instance|
|`ssh devopsteam09@15.188.43.46 -i ~/.ssh/CLD_KEY_DMZ_SSH_CLD_DEVOPSTEAM09.pem -L 2223:10.0.9.10:22`|connect to the subnet|
|`ssh -J cld_dmz bitnami@10.0.9.10 -i ~/.ssh/CLD_KEY_DRUPAL_DEVOPSTEAM09.pem`|connect to subnet with ssh|
|`EC2_PRIVATE_DRUPAL_DEVOPSTEAM09_WEEK03`|creat image|



### Deploy the elastic load balancer

In this task you will create a load balancer in AWS that will receive
the HTTP requests from clients and forward them to the Drupal
instances.

![Schema](./img/CLD_AWS_INFA.PNG)

## Task 01 Prerequisites for the ELB

* Create a dedicated security group

|Key|Value|
|:--|:--|
|Name|SG-DEVOPSTEAM[XX]-LB|
|Inbound Rules|Application Load Balancer|
|Outbound Rules|Refer to the infra schema|

```bash
[INPUT]
aws ec2 create-security-group \
 --group-name SG-DEVOPSTEAM09-LB \
 --description "SG-DEVOPSTEAM09-LB" \
 --vpc-id vpc-03d46c285a2af77ba \
 --tag-specifications "ResourceType=security-group,Tags=[{Key=Name,Value=SG-DEVOPSTEAM09-LB}]"

[OUTPUT]
{
    "GroupId": "sg-00c1b2f1908fb0a8c",
    "Tags": [
        {
            "Key": "Name",
            "Value": "SG-DEVOPSTEAM09-LB"
        }
    ]
}

[INPUT]
aws ec2 authorize-security-group-ingress \
 --group-id sg-00c1b2f1908fb0a8c \
 --ip-permissions IpProtocol=tcp,FromPort=8080,ToPort=8080,IpRanges='[{CidrIp=10.0.0.0/28}]'

[OUTPUT]
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0063cfa9f0faa8070",
            "GroupId": "sg-00c1b2f1908fb0a8c",
            "GroupOwnerId": "709024702237",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 8080,
            "ToPort": 8080,
            "CidrIpv4": "10.0.0.0/28"
        },
        {
            "SecurityGroupRuleId": "sgr-01e3dc94308790f2f",
            "GroupId": "sg-00c1b2f1908fb0a8c",
            "GroupOwnerId": "709024702237",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 8080,
            "ToPort": 8080,
            "CidrIpv4": "10.0.9.0/28"
        },
        {
            "SecurityGroupRuleId": "sgr-0d89917146e3d8e5b",
            "GroupId": "sg-00c1b2f1908fb0a8c",
            "GroupOwnerId": "709024702237",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 8080,
            "ToPort": 8080,
            "CidrIpv4": "10.0.9.128/28"
        }
    ]
}
```

* Create the Target Group

|Key|Value|
|:--|:--|
|Target type|Instances|
|Name|TG-DEVOPSTEAM[XX]|
|Protocol and port|Refer to the infra schema|
|Ip Address type|IPv4|
|VPC|Refer to the infra schema|
|Protocol version|HTTP1|
|Health check protocol|HTTP|
|Health check path|/|
|Port|Traffic port|
|Healthy threshold|2 consecutive health check successes|
|Unhealthy threshold|2 consecutive health check failures|
|Timeout|5 seconds|
|Interval|10 seconds|
|Success codes|200|

```bash
[INPUT]
aws elbv2 create-target-group \
    --name TG-DEVOPSTEAM09 \
    --protocol HTTP \
    --ip-address-type ipv4 \
    --target-type instance \
    --vpc-id vpc-03d46c285a2af77ba \
    --protocol-version HTTP1 \
    --health-check-protocol HTTP \
    --health-check-port 8080 \
    --health-check-path "/" \
    --port 8080 \
    --healthy-threshold-count 2 \
    --unhealthy-threshold-count 2 \
    --health-check-timeout-seconds 5 \
    --health-check-interval-seconds 10 \
    --matcher HttpCode=200

[OUTPUT]
{
    "TargetGroups": [
        {
            "TargetGroupArn": "arn:aws:elasticloadbalancing:eu-west-3:709024702237:targetgroup/TG-DEVOPSTEAM09/2852afb9633e3b23",
            "TargetGroupName": "TG-DEVOPSTEAM09",
            "Protocol": "HTTP",
            "Port": 8080,
            "VpcId": "vpc-03d46c285a2af77ba",
            "HealthCheckProtocol": "HTTP",
            "HealthCheckPort": "8080",
            "HealthCheckEnabled": true,
            "HealthCheckIntervalSeconds": 10,
            "HealthCheckTimeoutSeconds": 5,
            "HealthyThresholdCount": 2,
            "UnhealthyThresholdCount": 2,
            "HealthCheckPath": "/",
            "Matcher": {
                "HttpCode": "200"
            },
            "TargetType": "instance",
            "ProtocolVersion": "HTTP1",
            "IpAddressType": "ipv4"
        }
    ]
}
```


## Task 02 Deploy the Load Balancer

[Source](https://aws.amazon.com/elasticloadbalancing/)

* Create the Load Balancer

|Key|Value|
|:--|:--|
|Type|Application Load Balancer|
|Name|ELB-DEVOPSTEAM99|
|Scheme|Internal|
|Ip Address type|IPv4|
|VPC|Refer to the infra schema|
|Security group|Refer to the infra schema|
|Listeners Protocol and port|Refer to the infra schema|
|Target group|Your own target group created in task 01|

Provide the following answers (leave any
field not mentioned at its default value):

```bash
[INPUT]
aws elbv2 create-load-balancer \
  --name ELB-DEVOPSTEAM09 \
  --scheme internal \
  --ip-address-type ipv4 \
  --subnets subnet-0a8b06840338a7299 subnet-0f9df600cde330c7d \
  --security-group sg-00c1b2f1908fb0a8c

[OUTPUT]
{
    "LoadBalancers": [
        {
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:eu-west-3:709024702237:loadbalancer/app/ELB-DEVOPSTEAM09/4f15812df622c56a",
            "DNSName": "internal-ELB-DEVOPSTEAM09-1659191669.eu-west-3.elb.amazonaws.com",
            "CanonicalHostedZoneId": "Z3Q77PNBQS71R4",
            "CreatedTime": "2024-03-23T12:13:16.860000+00:00",
            "LoadBalancerName": "ELB-DEVOPSTEAM09",
            "Scheme": "internal",
            "VpcId": "vpc-03d46c285a2af77ba",
            "State": {
                "Code": "provisioning"
            },
            "Type": "application",
            "AvailabilityZones": [
                {
                    "ZoneName": "eu-west-3a",
                    "SubnetId": "subnet-0a8b06840338a7299",
                    "LoadBalancerAddresses": []
                },
                {
                    "ZoneName": "eu-west-3b",
                    "SubnetId": "subnet-0f9df600cde330c7d",
                    "LoadBalancerAddresses": []
                }
            ],
            "SecurityGroups": [
                "sg-00c1b2f1908fb0a8c"
            ],
            "IpAddressType": "ipv4"
        }
    ]
}

[INPUT]
aws elbv2 register-targets \
--target-group-arn arn:aws:elasticloadbalancing:eu-west-3:709024702237:targetgroup/TG-DEVOPSTEAM09/2852afb9633e3b23 \
--targets Id=i-05f9070d716bcc424 Id=i-0399a7f7ce00690a5

[OUTPUT]
None

[INPUT]
aws elbv2 create-listener \
--load-balancer-arn arn:aws:elasticloadbalancing:eu-west-3:709024702237:loadbalancer/app/ELB-DEVOPSTEAM09/4f15812df622c56a \
--default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:eu-west-3:709024702237:targetgroup/TG-DEVOPSTEAM09/2852afb9633e3b23 \
--protocol HTTP \
--port 8080

[OUTPUT]
{
    "Listeners": [
        {
            "ListenerArn": "arn:aws:elasticloadbalancing:eu-west-3:709024702237:listener/app/ELB-DEVOPSTEAM09/4f15812df622c56a/5eec1c933c3ade7d",
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:eu-west-3:709024702237:loadbalancer/app/ELB-DEVOPSTEAM09/4f15812df622c56a",
            "Port": 8080,
            "Protocol": "HTTP",
            "DefaultActions": [
                {
                    "Type": "forward",
                    "TargetGroupArn": "arn:aws:elasticloadbalancing:eu-west-3:709024702237:targetgroup/TG-DEVOPSTEAM09/2852afb9633e3b23",
                    "ForwardConfig": {
                        "TargetGroups": [
                            {
                                "TargetGroupArn": "arn:aws:elasticloadbalancing:eu-west-3:709024702237:targetgroup/TG-DEVOPSTEAM09/2852afb9633e3b23",
                                "Weight": 1
                            }
                        ],
                        "TargetGroupStickinessConfig": {
                            "Enabled": false
                        }
                    }
                }
            ]
        }
    ]
}
```

* Get the ELB FQDN (DNS NAME - A Record)

```bash
NEED HELP HERE

[INPUT]
aws elbv2 describe-load-balancers | \
   jq '.LoadBalancers[] | 
       select(.DNSName == "internal-ELB-DEVOPSTEAM09-1659191669.eu-west-3.elb.amazonaws.com") | 
       .LoadBalancerArn'

[OUTPUT]

```

* Get the ELB deployment status

Note : In the EC2 console select the Target Group. In the
       lower half of the panel, click on the **Targets** tab. Watch the
       status of the instance go from **unused** to **initial**.

* Ask the DMZ administrator to register your ELB with the reverse proxy via the private teams channel

* Update your string connection to test your ELB and test it

```bash
aws elbv2 describe-target-health --target-group-arn arn:aws:elasticloadbalancing:eu-west-3:709024702237:targetgroup/TG-DEVOPSTEAM09/2852afb9633e3b23
```

* Test your application through your ssh tunneling

```bash
[INPUT]
ssh devopsteam09@15.188.43.46 -i ~/.ssh/CLD_KEY_DMZ_DEVOPSTEAM09.pem -L 8080:internal-ELB-DEVOPSTEAM09-1659191669.eu-west-3.elb.amazonaws.com:8080 
Open localhost:8080 in your browser or curl localhost:8080 from another terminal

[OUTPUT]
The drupal home page.

```

#### Questions - Analysis

* On your local machine resolve the DNS name of the load balancer into
  an IP address using the `nslookup` command (works on Linux, macOS and Windows). Write
  the DNS name and the resolved IP Address(es) into the report.

```
nslookup internal-ELB-DEVOPSTEAM09-1659191669.eu-west-3.elb.amazonaws.com
Server:		10.193.64.16
Address:	10.193.64.16#53

Non-authoritative answer:
Name:	internal-ELB-DEVOPSTEAM09-1659191669.eu-west-3.elb.amazonaws.com
Address: 10.0.9.138
Name:	internal-ELB-DEVOPSTEAM09-1659191669.eu-west-3.elb.amazonaws.com
Address: 10.0.9.4
```

* From your Drupal instance, identify the ip from which requests are sent by the Load Balancer.

Help : execute `tcpdump port 8080`

```
bitnami@ip-10-0-9-140:~$ sudo tcpdump port 8080
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ens5, link-type EN10MB (Ethernet), snapshot length 262144 bytes
14:56:17.680962 IP 10.0.9.4.42598 > 10.0.9.140.http-alt: Flags [S], seq 3043169205, win 26883, options [mss 8961,sackOK,TS val 3489409212 ecr 0,nop,wscale 8], length 0
14:56:17.680988 IP 10.0.9.140.http-alt > 10.0.9.4.42598: Flags [S.], seq 3615836376, ack 3043169206, win 62643, options [mss 8961,sackOK,TS val 1603475500 ecr 3489409212,nop,wscale 7], length 0
14:56:17.681941 IP 10.0.9.4.42598 > 10.0.9.140.http-alt: Flags [.], ack 1, win 106, options [nop,nop,TS val 3489409213 ecr 1603475500], length 0
14:56:17.681966 IP 10.0.9.4.42598 > 10.0.9.140.http-alt: Flags [P.], seq 1:131, ack 1, win 106, options [nop,nop,TS val 3489409213 ecr 1603475500], length 130: HTTP: GET / HTTP/1.1
14:56:17.681984 IP 10.0.9.140.http-alt > 10.0.9.4.42598: Flags [.], ack 131, win 489, options [nop,nop,TS val 1603475501 ecr 3489409213], length 0
14:56:17.704503 IP 10.0.9.140.http-alt > 10.0.9.4.42598: Flags [P.], seq 1:5625, ack 131, win 489, options [nop,nop,TS val 1603475523 ecr 3489409213], length 5624: HTTP: HTTP/1.1 200 OK
14:56:17.704605 IP 10.0.9.140.http-alt > 10.0.9.4.42598: Flags [F.], seq 5625, ack 131, win 489, options [nop,nop,TS val 1603475523 ecr 3489409213], length 0
14:56:17.705449 IP 10.0.9.4.42598 > 10.0.9.140.http-alt: Flags [.], ack 5625, win 175, options [nop,nop,TS val 3489409236 ecr 1603475523], length 0
14:56:17.705531 IP 10.0.9.4.42598 > 10.0.9.140.http-alt: Flags [F.], seq 131, ack 5626, win 175, options [nop,nop,TS val 3489409236 ecr 1603475523], length 0
14:56:17.705538 IP 10.0.9.140.http-alt > 10.0.9.4.42598: Flags [.], ack 132, win 489, options [nop,nop,TS val 1603475524 ecr 3489409236], length 0
14:56:21.176682 IP 10.0.9.138.25462 > 10.0.9.140.http-alt: Flags [S], seq 221292961, win 26883, options [mss 8961,sackOK,TS val 404465218 ecr 0,nop,wscale 8], length 0
14:56:21.176711 IP 10.0.9.140.http-alt > 10.0.9.138.25462: Flags [S.], seq 1778857856, ack 221292962, win 62643, options [mss 8961,sackOK,TS val 3314245021 ecr 404465218,nop,wscale 7], length 0
14:56:21.176882 IP 10.0.9.138.25462 > 10.0.9.140.http-alt: Flags [.], ack 1, win 106, options [nop,nop,TS val 404465218 ecr 3314245021], length 0
14:56:21.176883 IP 10.0.9.138.25462 > 10.0.9.140.http-alt: Flags [P.], seq 1:131, ack 1, win 106, options [nop,nop,TS val 404465218 ecr 3314245021], length 130: HTTP: GET / HTTP/1.1
14:56:21.176907 IP 10.0.9.140.http-alt > 10.0.9.138.25462: Flags [.], ack 131, win 489, options [nop,nop,TS val 3314245022 ecr 404465218], length 0
14:56:21.206854 IP 10.0.9.140.http-alt > 10.0.9.138.25462: Flags [P.], seq 1:5625, ack 131, win 489, options [nop,nop,TS val 3314245052 ecr 404465218], length 5624: HTTP: HTTP/1.1 200 OK
14:56:21.206924 IP 10.0.9.140.http-alt > 10.0.9.138.25462: Flags [F.], seq 5625, ack 131, win 489, options [nop,nop,TS val 3314245052 ecr 404465218], length 0
14:56:21.206955 IP 10.0.9.138.25462 > 10.0.9.140.http-alt: Flags [.], ack 5625, win 175, options [nop,nop,TS val 404465248 ecr 3314245052], length 0
14:56:21.207037 IP 10.0.9.138.25462 > 10.0.9.140.http-alt: Flags [F.], seq 131, ack 5626, win 175, options [nop,nop,TS val 404465248 ecr 3314245052], length 0
14:56:21.207044 IP 10.0.9.140.http-alt > 10.0.9.138.25462: Flags [.], ack 132, win 489, options [nop,nop,TS val 3314245052 ecr 404465248], length 0
```

* In the Apache access log identify the health check accesses from the
  load balancer and copy some samples into the report.

```
bitnami@ip-10-0-9-140:~$ tail -n 10 /opt/bitnami/apache2/logs/access_log
10.0.9.4 - - [28/Mar/2024:14:56:17 +0000] "GET / HTTP/1.1" 200 5149
10.0.9.138 - - [28/Mar/2024:14:56:21 +0000] "GET / HTTP/1.1" 200 5149
10.0.9.4 - - [28/Mar/2024:14:56:27 +0000] "GET / HTTP/1.1" 200 5149
10.0.9.138 - - [28/Mar/2024:14:56:31 +0000] "GET / HTTP/1.1" 200 5149
10.0.9.4 - - [28/Mar/2024:14:56:37 +0000] "GET / HTTP/1.1" 200 5149
10.0.9.138 - - [28/Mar/2024:14:56:41 +0000] "GET / HTTP/1.1" 200 5149
10.0.9.4 - - [28/Mar/2024:14:56:47 +0000] "GET / HTTP/1.1" 200 5149
10.0.9.138 - - [28/Mar/2024:14:56:51 +0000] "GET / HTTP/1.1" 200 5149
10.0.9.4 - - [28/Mar/2024:14:56:57 +0000] "GET / HTTP/1.1" 200 5149
10.0.9.138 - - [28/Mar/2024:14:57:01 +0000] "GET / HTTP/1.1" 200 5149
```

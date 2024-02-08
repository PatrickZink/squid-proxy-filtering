# squid-proxy-filtering

## Description 

The CloudFormation template provided deploys Squid proxy instances across each Availability Zone (AZ) within a Virtual Private Cloud (VPC). This configuration creates a "transparent proxy" capable of restricting both HTTP and HTTPS outbound traffic to a specified set of internet domains, while maintaining complete transparency for instances within the private subnet. It is designed to automatically route traffic from the private subnets through the Elastic Network Interface (ENI) associated with the corresponding Squid proxy. The Squid proxies are maintained within an Auto Scaling Group, ensuring scalability and reliability, and their health is monitored by CloudWatch Alarms. If a Squid proxy becomes unavailable, the default network route for the affected AZ is automatically redirected to another operational instance. Furthermore, updates to the proxy whitelist can be easily made by modifying the CloudFormation stack parameters.

![squid-proxy drawio](https://github.com/PatrickZink/squid-proxy-filtering/assets/70896863/5d1c173a-3df9-4fad-8e60-c5ff40032959)


## Installation

To deploy the stack, the following prerequisites must be met:
- A VPC with at least one private and one public subnet is available.
- The public subnet has an attached Internet Gateway (IGW).
- The public subnet has a route to the IGW for 0.0.0.0/0.
- If a NAT Gateway is already in place, it can be deleted. Since the stack routes the 0.0.0.0/0 traffic to the Squid Proxy, no NAT Gateway is necessary.

Deploy the stack squid-proxy-filtering.yml using CloudFormation as a new stack. Fill out the input parameters. The domain list can be entered with each domain separated by a newline. If there are multiple private subnets, they can be specified separated by commas. If only one or two Availability Zones (AZs) are present, the remaining PrivateRouteTables and PublicSubnet fields can be left blank.

![image](https://github.com/PatrickZink/squid-proxy-filtering/assets/70896863/fb02ba36-2487-4385-8a0f-2e412bde55d6)

## Outbound traffic test

For testing purposes, one can access websites from an EC2 instance within the private network to validate the whitelist functionality. Execute a request to a site that is on the whitelist to see it succeed, and contrast this with a request to a non-whitelisted site to observe the expected failure.

```curl https://www.amazon.com``` - This should work as it's on the whitelist. 

![image](https://github.com/PatrickZink/squid-proxy-filtering/assets/70896863/e71be4fd-f960-454b-995b-176213a362e6)


```curl https://www.wikipedia.com``` - This should fail because it's not on the whitelist.

![image](https://github.com/PatrickZink/squid-proxy-filtering/assets/70896863/a45ea510-27e0-439b-8c9d-592a208161df)


## CloudWatch Logs

The access logs for all Squid Proxies can be viewed in CloudWatch under the LogGroups section at /filtering-nat-instance/access.log. 

![image](https://github.com/PatrickZink/squid-proxy-filtering/assets/70896863/7e9aee77-b284-4325-b9b9-f78114d29444)

With CloudWatch Insights, SQL-like queries can be executed across all proxy logs, which allows for the analysis of domain access and blocked requests.

```fields @timestamp, @message filter @message like 'TCP_DENIED'```

![image](https://github.com/PatrickZink/squid-proxy-filtering/assets/70896863/b0899a24-97a0-4646-aaac-3f6f41824031)

## Allowed Domain List update

The Allowed Domain List is specified or modified during the deployment of the CloudFormation template. To update the Allowed Domain List, the CloudFormation stack needs to be updated with the new list provided as an input parameter. Subsequently, the Squid proxies will automatically update their configuration with the revised settings.



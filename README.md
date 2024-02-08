# squid-proxy-filtering

## Description 

This CloudFormation template deploys a Squid proxy instance in each AZ within a VPC. It automatically routes traffic from the private subnets to the Elastic Network Interface (ENI) of the respective Squid proxy. The Squid proxies are managed within an Auto Scaling Group, and CloudWatch Alarms monitor their health. If a Squid proxy becomes unavailable, the default route for the corresponding AZ is redirected to another healthy instance. Adjustments to the proxy whitelist can be made directly by updating the CloudFormation stack parameters.

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



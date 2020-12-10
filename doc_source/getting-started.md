# Getting started with Gateway Load Balancers<a name="getting-started"></a>

Gateway Load Balancers make it easy to deploy, scale, and manage third\-party virtual appliances, such as security appliances\.

In this tutorial, we'll implement an inspection system using a Gateway Load Balancer and a Gateway Load Balancer endpoint\.

**Topics**
+ [Overview](#overview)
+ [Prerequisites](#prerequisites)
+ [Step 1: Create a Gateway Load Balancer and register targets](#create-register)
+ [Step 2: Create a Gateway Load Balancer endpoint](#create-endpoint)
+ [Step 3: Configure routing](#configure-routing)

## Overview<a name="overview"></a>

The security appliances run in a subnet of the service provider VPC\. The Gateway Load Balancer is in another subnet of the service provider VPC\.

The application servers run in a subnet of the service consumer VPC\. The network interface for the Gateway Load Balancer endpoint is in another subnet of the service consumer VPC\. All traffic entering the service consumer VPC through the internet gateway is routed to the Gateway Load Balancer endpoint for inspection before it's routed to the destination subnet\. Similarly, all traffic leaving the EC2 instance is routed to the Gateway Load Balancer endpoint for inspection before it's routed to the internet\.

![\[Using a Gateway Load Balancer endpoint to access an endpoint service\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/images/vpc-endpoint-service-gwlbe.png)

### Routing<a name="route-tables"></a>

The route table for the internet gateway must have an entry that routes traffic destined for the application servers to the Gateway Load Balancer endpoint\. To specify the Gateway Load Balancer endpoint, use the ID of the VPC endpoint\.


| Destination | Target | 
| --- | --- | 
| 10\.0\.0\.0/16 | Local | 
| 10\.0\.1\.0/24 | vpc\-endpoint\-id | 

The route table for the subnet with the application servers must have an entry that routes all traffic \(0\.0\.0\.0/0\) from the application servers to the Gateway Load Balancer endpoint\.


| Destination | Target | 
| --- | --- | 
| 10\.0\.0\.0/16 | Local | 
| 0\.0\.0\.0/0 | vpc\-endpoint\-id | 

The route table for the subnet with the Gateway Load Balancer endpoint must route traffic that returns from inspection to its final destination\. For traffic that originated from the internet, the local route ensures that it reaches the application servers\. For traffic that originated from the application servers, add an entry that routes all traffic \(0\.0\.0\.0/0\) to the internet gateway\.


| Destination | Target | 
| --- | --- | 
| 10\.0\.0\.0/16 | Local | 
| 0\.0\.0\.0/0 | internet\-gateway\-id | 

## Prerequisites<a name="prerequisites"></a>
+ Ensure that the service consumer VPC has at least two subnets for each Availability Zone that contains application servers\. One subnet is for the Gateway Load Balancer endpoint, and the other is for the application servers\.
+ Ensure that the service provider VPC has at least two subnets for each Availability Zone that contains security appliance instances\. One subnet is for the Gateway Load Balancer, and the other is for the instances\.
+ Launch at least one security appliance instance in each security appliance subnet in the service provider VPC\. The security groups for these instances must allow UDP traffic on port 6081\.

## Step 1: Create a Gateway Load Balancer and register targets<a name="create-register"></a>

Use the following procedure to create your load balancer, listener, and target group, and to register your security appliance instances as targets\.

**To create a Gateway Load Balancer and register targets**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Choose **Create Load Balancer**\.

1. For **Gateway Load Balancer**, choose **Create**\.

1. For **Name**, enter a name for your load balancer\. For example, **my\-load\-balancer**\.

1. For **Availability Zones**, select the service provider VPC\. For each Availability Zone in which you launched security appliance instances, select the Availability Zone and then select a public subnet\.

1. \(Optional\) Expand **Tags** and add tags\.

1. Choose **Next: Configure Routing**\.

1. For **Name**, enter a name for your target group\. For example, **my\-targets**\.

1. For **Target type**, select `instance` to specify targets by instance ID or `ip` to specify targets by IP address\.

1. **Protocol** must be `GENEVE`, and **Port** must be `6081`\.

1. \(Optional\) For **Health checks**, modify the health check settings as needed\.

1. Choose **Next: Register Targets**\.

1. Add your instances or IP addresses to the list, and then choose **Next: Review**\.

1. Choose **Create**\.

## Step 2: Create a Gateway Load Balancer endpoint<a name="create-endpoint"></a>

Use the following procedure to create a Gateway Load Balancer endpoint\. Gateway Load Balancer endpoints are zonal\. We recommend that you create one Gateway Load Balancer endpoint per zone\. For more information, see [Gateway Load Balancer endpoints \(AWS PrivateLink\)](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-gateway-load-balancer.html)\.

**To create a Gateway Load Balancer endpoint**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Endpoint Services**\.

1. Choose **Create Endpoint Service** and do the following:

   1. For **Associate Load Balancers**, select your Gateway Load Balancer\.

   1. For **Require acceptance for endpoint**, select **Acceptance required** to accept connection requests to your service manually\. Otherwise, endpoint connections are automatically accepted\.

   1. \(Optional\) To add a tag, choose **Add tag** and then specify the key and value for the tag\.

   1. Choose **Create service**\. Choose the service ID\. Save the service name from the **Details** tab; you'll need it when you create the endpoint\.

   1. Choose **Actions**, **Add principals to whitelist**\. Enter the ARNs of the service consumers that are allowed to create an endpoint to your service\. A service consumer can be an IAM user, IAM role, or AWS account\.

1. In the navigation pane, choose **Endpoints**\.

1. Choose **Create Endpoint** and do the following:

   1. For **Service category**, choose **Find service by name**\.

   1. For **Service name**, enter the service name that you saved earlier, and then choose **Verify**\. If the name is found, proceed to the next step\. Otherwise, be sure that you used the correct service name\.

   1. For **VPC**, select the service consumer VPC\.

   1. For **Subnets**, select a subnet for the Gateway Load Balancer endpoint\.

   1. \(Optional\) To add a tag, choose **Add tag** and specify the key and value for the tag\.

   1. Choose **Create endpoint**\. The initial status is `pending acceptance`\.

## Step 3: Configure routing<a name="configure-routing"></a>

Configure the route tables for the service consumer VPC as follows\. This allows the security appliances to perform security inspection on inbound traffic that's destined for the application servers\.

**To configure routing**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Route Tables**\.

1. Select the route table for the internet gateway and do the following:

   1. Choose **Actions**, **Edit routes**\.

   1. Choose **Add route**\. For **Destination**, enter the CIDR block of the subnet for the application servers \(for example, 10\.0\.1\.0/24\)\. For **Target**, select the VPC endpoint\.

   1. Choose **Save routes**\.

1. Select the route table for the subnet with the application servers and do the following:

   1. Choose **Actions**, **Edit routes**\.

   1. Choose **Add route**\. For **Destination**, enter **0\.0\.0\.0/0**\. For **Target**, select the VPC endpoint\.

   1. Choose **Save routes**\.

1. Select the route table for the subnet with the Gateway Load Balancer endpoint, and do the following:

   1. Choose **Actions**, **Edit routes**\.

   1. Choose **Add route**\. For **Destination**, enter **0\.0\.0\.0/0**\. For **Target**, select the internet gateway\.

   1. Choose **Save routes**\.
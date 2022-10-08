# Getting started with Gateway Load Balancers<a name="getting-started"></a>

Gateway Load Balancers make it easy to deploy, scale, and manage third\-party virtual appliances, such as security appliances\.

In this tutorial, we'll implement an inspection system using a Gateway Load Balancer and a Gateway Load Balancer endpoint\.

**Topics**
+ [Overview](#overview)
+ [Prerequisites](#prerequisites)
+ [Step 1: Register targets and create a Gateway Load Balancer](#create-register)
+ [Step 2: Create a Gateway Load Balancer endpoint service](#create-endpoint-service)
+ [Step 3: Create a Gateway Load Balancer endpoint](#create-endpoint)
+ [Step 4: Configure routing](#configure-routing)

## Overview<a name="overview"></a>

A Gateway Load Balancer endpoint is a VPC endpoint that provides private connectivity between virtual appliances in the service provider VPC, and application servers in the service consumer VPC\. The Gateway Load Balancer is deployed in the same VPC as that of the virtual appliances\. These appliances are registered as a target group of the Gateway Load Balancer\.

The application servers run in one subnet \(destination subnet\) in the service consumer VPC, while the Gateway Load Balancer endpoint is in another subnet of the same VPC\. All traffic entering the service consumer VPC through the internet gateway is first routed to the Gateway Load Balancer endpoint for inspection and then routed to the destination subnet\.

Similarly, all traffic leaving the application servers \(destination subnet\) is routed to the Gateway Load Balancer endpoint for inspection before it is routed back to the internet\. The following network diagram is a visual representation of how a Gateway Load Balancer endpoint is used to access an endpoint service\.

![\[Using a Gateway Load Balancer endpoint to access an endpoint service\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/images/vpc-endpoint-service-gwlbe-new.png)

The numbered items that follow, highlight and explain elements shown in the preceding image\. 

**Traffic from the internet to the application \(blue arrows\):**

1. Traffic enters the service consumer VPC through the internet gateway\.

1. Traffic is sent to the Gateway Load Balancer endpoint, as a result of ingress routing\.

1. Traffic is sent to the Gateway Load Balancer for inspection through the security appliance\.

1. Traffic is sent back to the Gateway Load Balancer endpoint after inspection\.

1. Traffic is sent to the application servers \(destination subnet\)\.

**Traffic from the application to the internet \(orange arrows\):**

1. Traffic is sent to the Gateway Load Balancer endpoint as a result of the default route configured on the application server subnet\.

1. Traffic is sent to the Gateway Load Balancer for inspection through the security appliance\.

1.  Traffic is sent back to the Gateway Load Balancer endpoint after inspection\. 

1. Traffic is sent to the internet gateway based on the route table configuration\.

1. Traffic is routed back to the internet\.

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
+ The Gateway Load Balancer and the targets can be in the same subnet\.
+ You cannot use a subnet that is shared from another account to deploy the Gateway Load Balancer\.
+ Launch at least one security appliance instance in each security appliance subnet in the service provider VPC\. The security groups for these instances must allow UDP traffic on port 6081\.

## Step 1: Register targets and create a Gateway Load Balancer<a name="create-register"></a>

Use the following procedure to create your target group, register your security appliance instances as targets, and then create your load balancer and listener\.

**To create a target group and register targets**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Load Balancing**, choose **Target Groups**\.

1. Choose **Create target group**\.

1. For **Choose a target type**, select **Instances** to specify targets by instance ID, or **IP addresses** to specify targets by IP address\.

1. For **Target group name**, enter a name for your target group\. For example, **my\-targets**\.

1. **Protocol** must be `GENEVE`, and **Port** must be `6081`\. No other protocols or ports are supported\.

1. For **VPC**, select a virtual private cloud \(VPC\) with the instances to include in the target group\.

1. \(Optional\) For **Health checks**, modify the health check settings as needed\.

1. \(Optional\) Expand **Tags** and add tags\.

1. Choose **Next**\.

1. Add one or more targets as follows:
   + If the target type is **Instances**, select one or more instances, enter one or more ports, and then choose **Include as pending below**\.
   + If the target type is **IP addresses**, select the network, enter the IP address and ports, and then choose **Include as pending below**\.

1. Choose **Create target group**\.

**To create a Gateway Load Balancer**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **Load Balancing**, choose **Load Balancers**\.

1. Choose **Create Load Balancer**\.

1. Under **Gateway Load Balancer**, choose **Create**\.

1. For **Load balancer name**, enter a name for your load balancer\. For example, **my\-glb**\. 

1. For **IP address type**, you must choose **IPv4**, because your clients can only use IPv4 addresses to communicate with the load balancer\.

1. For **VPC**, select the service provider VPC\.

1. For **Mappings**, select all of the Availability Zones in which you launched security appliance instances, and the corresponding public subnets\.

1. For **Default action**, select a target group to forward traffic to\. If you don't have a target group, create one first\. The target group must use the GENEVE protocol\.

1. \(Optional\) Expand **Tags** and add tags\.

1. Review your configuration, and then choose **Create load balancer**\.

## Step 2: Create a Gateway Load Balancer endpoint service<a name="create-endpoint-service"></a>

Use the following procedure to create an endpoint service using a Gateway Load Balancer\.

**To create a Gateway Load Balancer endpoint service**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Endpoint services**\.

1. Choose **Create endpoint service** and do the following:

   1. For **Load balancer type**, choose **Gateway**\.

   1. For **Available load balancers**, select your Gateway Load Balancer\.

   1. For **Require acceptance for endpoint**, select **Acceptance required** to accept connection requests to your service manually\. Otherwise, they are automatically accepted\.

   1. \(Optional\) To add a tag, choose **Add new tag** and enter the tag key and tag value\.

   1. Choose **Create**\. Note the service name; you'll need it when you create the endpoint\.

1. Select the new endpoint service and choose **Actions**, **Allow principals**\. Enter the ARNs of the service consumers that are allowed to create an endpoint to your service\. A service consumer can be an IAM user, IAM role, or AWS account\. Choose **Allow principals**\.

## Step 3: Create a Gateway Load Balancer endpoint<a name="create-endpoint"></a>

Use the following procedure to create a Gateway Load Balancer endpoint\. Gateway Load Balancer endpoints are zonal\. We recommend that you create one Gateway Load Balancer endpoint per zone\. For more information, see [Access virtual appliances through AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/privatelink/vpce-gateway-load-balancer.html) in the *AWS PrivateLink Guide*\.

**To create a Gateway Load Balancer endpoint**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Endpoints**\.

1. Choose **Create endpoint** and do the following:

   1. For **Service category**, choose **Other endpoint services**\.

   1. For **Service name**, enter the service name that you noted earlier, and then choose **Verify service**\.

   1. For **VPC**, select the service consumer VPC\.

   1. For **Subnets**, select a subnet for the Gateway Load Balancer endpoint\.

   1. \(Optional\) To add a tag, choose **Add new tag** and enter the tag key and tag value\.

   1. Choose **Create endpoint**\. The initial status is `pending acceptance`\.

To accept the endpoint connection request, use the following procedure\.

1. In the navigation pane, choose **Endpoint services**\.

1. Select the endpoint service\.

1. From the **Endpoint connections** tab, select the endpoint connection\.

1. To accept the connection request, choose **Actions**, **Accept endpoint connection request**\. When prompted for confirmation, enter **accept** and then choose **Accept**\.

## Step 4: Configure routing<a name="configure-routing"></a>

Configure the route tables for the service consumer VPC as follows\. This allows the security appliances to perform security inspection on inbound traffic that's destined for the application servers\.

**To configure routing**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Route tables**\.

1. Select the route table for the internet gateway and do the following:

   1. Choose **Actions**, **Edit routes**\.

   1. Choose **Add route**\. For **Destination**, enter the CIDR block of the subnet for the application servers \(for example, 10\.0\.1\.0/24\)\. For **Target**, select the VPC endpoint\.

   1. Choose **Save changes**\.

1. Select the route table for the subnet with the application servers and do the following:

   1. Choose **Actions**, **Edit routes**\.

   1. Choose **Add route**\. For **Destination**, enter **0\.0\.0\.0/0**\. For **Target**, select the VPC endpoint\.

   1. Choose **Save changes**\.

1. Select the route table for the subnet with the Gateway Load Balancer endpoint, and do the following:

   1. Choose **Actions**, **Edit routes**\.

   1. Choose **Add route**\. For **Destination**, enter **0\.0\.0\.0/0**\. For **Target**, select the internet gateway\.

   1. Choose **Save changes**\.
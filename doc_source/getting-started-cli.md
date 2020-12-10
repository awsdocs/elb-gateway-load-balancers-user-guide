# Getting started with Gateway Load Balancers using the AWS CLI<a name="getting-started-cli"></a>

Gateway Load Balancers make it easy to deploy, scale, and manage third\-party virtual appliances, such as security appliances\.

In this tutorial, we'll implement an inspection system using a Gateway Load Balancer and a Gateway Load Balancer endpoint\.

**Topics**
+ [Overview](#overview-cli)
+ [Prerequisites](#prerequisites-aws-cli)
+ [Step 1: Create a Gateway Load Balancer and register targets](#create-load-balancer-aws-cli)
+ [Step 2: Create a Gateway Load Balancer endpoint](#create-endpoint-aws-cli)
+ [Step 3: Configure routing](#configure-routing-aws-cli)

## Overview<a name="overview-cli"></a>

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

## Prerequisites<a name="prerequisites-aws-cli"></a>
+ Install the AWS CLI or update to the current version of the AWS CLI if you are using a version that does not support Gateway Load Balancers\. For more information, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*\.
+ Ensure that the service consumer VPC has at least two subnets for each Availability Zone that contains application servers\. One subnet is for the Gateway Load Balancer endpoint, and the other is for the application servers\.
+ Ensure that the service provider VPC has at least two subnets for each Availability Zone that contains security appliance instances\. One subnet is for the Gateway Load Balancer, and the other is for the instances\.
+ Launch at least one security appliance instance in each security appliance subnet in the service provider VPC\. The security groups for these instances must allow UDP traffic on port 6081\.

## Step 1: Create a Gateway Load Balancer and register targets<a name="create-load-balancer-aws-cli"></a>

Use the following procedure to create your load balancer, listener, and target groups, and to register your security appliance instances as targets\.

**To create a Gateway Load Balancer and register targets**

1. Use the [create\-load\-balancer](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-load-balancer.html) command to create a load balancer of type `gateway`\. You can specify one subnet for each Availability Zone in which you launched security appliance instances\.

   ```
   aws elbv2 create-load-balancer --name my-load-balancer --type gateway --subnets provider-subnet-id
   ```

   The output includes the Amazon Resource Name \(ARN\) of the load balancer, with the format shown in the following example\.

   ```
   arn:aws:elasticloadbalancing:us-east-2:123456789012:loadbalancer/gwy/my-load-balancer/1234567890123456
   ```

1. Use the [create\-target\-group](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-target-group.html) command to create a target group, specifying the service provider VPC in which you launched your instances\.

   ```
   aws elbv2 create-target-group --name my-targets --protocol GENEVE --port 6081 --vpc-id provider-vpc-id
   ```

   The output includes the ARN of the target group, with the following format\.

   ```
   arn:aws:elasticloadbalancing:us-east-2:123456789012:targetgroup/my-targets/0123456789012345
   ```

1. Use the [register\-targets](https://docs.aws.amazon.com/cli/latest/reference/elbv2/register-targets.html) command to register your instances with your target group\.

   ```
   aws elbv2 register-targets --target-group-arn targetgroup-arn --targets Id=i-1234567890abcdef0 Id=i-0abcdef1234567890
   ```

1. Use the [create\-listener](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-listener.html) command to create a listener for your load balancer with a default rule that forwards requests to your target group\.

   ```
   aws elbv2 create-listener --load-balancer-arn loadbalancer-arn --default-actions Type=forward,TargetGroupArn=targetgroup-arn
   ```

   The output contains the ARN of the listener, with the following format\.

   ```
   arn:aws:elasticloadbalancing:us-east-2:123456789012:listener/gwy/my-load-balancer/1234567890123456/abc1234567890123
   ```

1. \(Optional\) You can verify the health of the registered targets for your target group using the following [describe\-target\-health](https://docs.aws.amazon.com/cli/latest/reference/elbv2/describe-target-health.html) command\.

   ```
   aws elbv2 describe-target-health --target-group-arn targetgroup-arn
   ```

## Step 2: Create a Gateway Load Balancer endpoint<a name="create-endpoint-aws-cli"></a>

Use the following procedure to create a Gateway Load Balancer endpoint\. Gateway Load Balancer endpoints are zonal\. We recommend that you create one Gateway Load Balancer endpoint per zone\. For more information, see [Gateway Load Balancer endpoints \(AWS PrivateLink\)](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-gateway-load-balancer.html)\.

**To create a Gateway Load Balancer endpoint**

1. Use the [create\-vpc\-endpoint\-service\-configuration](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-vpc-endpoint-service-configuration.html) command to create an endpoint service configuration using your Gateway Load Balancer\.

   ```
   aws ec2 create-vpc-endpoint-service-configuration --gateway-load-balancer-arns loadbalancer-arn --no-acceptance-required
   ```

   The output contains the service ID \(for example, vpce\-svc\-12345678901234567\) and the service name \(for example, com\.amazonaws\.vpce\.us\-east\-2\.vpce\-svc\-12345678901234567\)\.

1. Use the [modify\-vpc\-endpoint\-service\-permissions](https://docs.aws.amazon.com/cli/latest/reference/ec2/modify-vpc-endpoint-service-permissions.html) command to allow service consumers to create an endpoint to your service\. A service consumer can be an IAM user, IAM role, or AWS account\. The following example adds permission for the specified AWS account\.

   ```
   aws ec2 modify-vpc-endpoint-service-permissions --service-id vpce-svc-12345678901234567 --add-allowed-principals arn:aws:iam::123456789012:root
   ```

1. Use the [create\-vpc\-endpoint](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-vpc-endpoint.html) command to create the Gateway Load Balancer endpoint for your service\.

   ```
   aws ec2 create-vpc-endpoint --vpc-endpoint-type GatewayLoadBalancer --service-name com.amazonaws.vpce.us-east-2.vpce-svc-12345678901234567 --vpc-id consumer-vpc-id --subnet-ids consumer-subnet-id
   ```

   The output contains the ID of the Gateway Load Balancer endpoint \(for example, vpce\-01234567890abcdef\)\.

## Step 3: Configure routing<a name="configure-routing-aws-cli"></a>

Configure the route tables for the service consumer VPC as follows\. This allows the security appliances to perform security inspection on inbound traffic that's destined for the application servers\.

**To configure routing**

1. Use the [create\-route](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-route.html) command to add an entry to the route table for the internet gateway that routes traffic that's destined for the application servers to the Gateway Load Balancer endpoint\.

   ```
   aws ec2 create-route --route-table-id gateway-rtb --destination-cidr-block 10.0.1.0/24 --vpc-endpoint-id vpce-01234567890abcdef
   ```

1. Use the [create\-route](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-route.html) command to add an entry to the route table for the subnet with the application servers that routes all traffic from the application servers to the Gateway Load Balancer endpoint\.

   ```
   aws ec2 create-route --route-table-id application-rtb --destination-cidr-block 0.0.0.0/0 --vpc-endpoint-id vpce-01234567890abcdef
   ```

1. Use the [create\-route](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-route.html) command to add an entry to the route table for the subnet with the Gateway Load Balancer endpoint that routes all traffic that originated from the application servers to the internet gateway\.

   ```
   aws ec2 create-route --route-table-id endpoint-rtb --destination-cidr-block 0.0.0.0/0 --gateway-id igw-01234567890abcdef
   ```

1. Repeat for each application subnet route table in each zone\.
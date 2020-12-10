# Create a Gateway Load Balancer<a name="create-load-balancer"></a>

A Gateway Load Balancer takes requests from clients and distributes them across targets in a target group, such as EC2 instances\.

Before you begin, ensure that the virtual private cloud \(VPC\) for your Gateway Load Balancer has at least one subnet in each Availability Zone where you have targets\.

To create a Gateway Load Balancer using the AWS CLI, see [Getting started using the CLI](getting-started-cli.md)\.

To create a Gateway Load Balancer using the AWS Management Console, complete the following tasks\.

**Topics**
+ [Step 1: Configure a load balancer and a listener](#configure-load-balancer)
+ [Step 2: Configure a target group](#configure-target-group)
+ [Step 3: Register targets with the target group](#select-targets)
+ [Step 4: Create the load balancer](#create-load-balancer)

## Step 1: Configure a load balancer and a listener<a name="configure-load-balancer"></a>

First, provide some basic configuration information for your Gateway Load Balancer, such as a name, and a network\. The listener for the load balancer listens for all IP packets across all ports\.

**To configure your load balancer and listener**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Choose **Create Load Balancer**\.

1. For **Gateway Load Balancer**, choose **Create**\.

1. For **Name**, enter a name for your Gateway Load Balancer\. For example, **my\-load\-balancer**\.

1. For **Availability Zones**, select the VPC that you used for your appliance instances\. For each Availability Zone that you used to launch your instances, select an Availability Zone and then select the subnet for that Availability Zone\.

1. \(Optional\) Expand **Tags** and add tags\.

1. Choose **Next: Configure Routing**\.

## Step 2: Configure a target group<a name="configure-target-group"></a>

You register targets, such as EC2 instances, with a target group\. The target group that you configure in this step is used as the target group in the listener rule, which forwards requests to the target group\. For more information, see [Target groups](target-groups.md)\.

**To configure your target group**

1. For **Target group**, keep the default, **New target group**\.

1. For **Name**, enter a name for the target group\.

1. For **Target type**, select `instance` to specify targets by instance ID or `ip` to specify targets by IP address\.

1. **Protocol** must be `GENEVE`, and **Port** must be `6081`\.

1. \(Optional\) For **Health checks**, modify the health check settings as needed\.

1. Choose **Next: Register Targets**\.

## Step 3: Register targets with the target group<a name="select-targets"></a>

You can register EC2 instances as targets in a target group\. The target type of the target group determines how you register targets with that target group\.

**To register targets by instance ID**

1. For **Instances**, select one or more instances and choose **Add to registered**\.

1. When you have finished adding instances to the list, choose **Next: Review**\.

**To register targets by IP address**

1. For each IP address to register, do the following:

   1. For **Network**, if the IP address is from a subnet of the target group VPC, select the VPC\. Otherwise, select **Other private IP address**\.

   1. For **IP**, enter the address\.

   1. Choose **Add to list**\.

1. When you have finished adding IP addresses to the list, choose **Next: Review**\.

## Step 4: Create the load balancer<a name="create-load-balancer"></a>

After creating your load balancer, you can verify that your EC2 instances have passed the initial health check and then test that the load balancer is sending traffic to your EC2 instances\. When you are finished with your load balancer, you can delete it\. For more information, see [Delete a load balancer](delete-load-balancer.md)\.

**To create the load balancer**

1. On the **Review** page, choose **Create**\.

1. After the load balancer is created, choose **Close**\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select the newly created target group\.

1. Choose **Targets** and verify that your instances are ready\. If the status of an instance is `initial`, it's probably because the instance is still in the process of being registered, or it has not passed the minimum number of health checks to be considered healthy\. After the status of at least one instance is healthy, you can test your load balancer\.
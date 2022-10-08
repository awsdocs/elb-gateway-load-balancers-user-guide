# Create a Gateway Load Balancer<a name="create-load-balancer"></a>

A Gateway Load Balancer takes requests from clients and distributes them across targets in a target group, such as EC2 instances\.

Before you begin, ensure that the virtual private cloud \(VPC\) for your Gateway Load Balancer has at least one subnet in each Availability Zone where you have targets\.

To create a Gateway Load Balancer using the AWS CLI, see [Getting started using the CLI](getting-started-cli.md)\.

To create a Gateway Load Balancer using the AWS Management Console, complete the following tasks\.

**Topics**
+ [Step 1: Configure your target group and register targets](#configure-target-group)
+ [Step 2: Configure the load balancer and listener](#configure-load-balancer)
+ [Important next steps](#important-next-steps)

## Step 1: Configure your target group and register targets<a name="configure-target-group"></a>

You can register targets, such as EC2 instances, with a target group\. You'll use the target group that you configure in this step when you configure your load balancer in the next step\. For more information, see [Target groups](target-groups.md)\.

**To configure your target group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **Load Balancing**, choose **Target Groups**\.

1. Choose **Create target group**\.

1. **Basic configuration**

   1. For **Choose a target type**, select **Instances** to specify targets by instance ID, or select **IP addresses** to specify targets by IP address\.

   1. For **Target group name**, enter a name for the target group\.

   1. Verify that **Protocol** is `GENEVE` and **Port** is `6081`\. No other protocols or ports are supported\.

   1. For **VPC**, select a virtual private cloud \(VPC\) with the instances to include in your target group\.

1. \(Optional\) For **Health checks**, modify the settings and advanced settings as needed\. If health checks consecutively exceed the **Unhealthy threshold** count, the load balancer takes the target out of service\. If health checks consecutively exceed the **Healthy threshold** count, the load balancer puts the target back in service\. For more information, see [Health checks for your target groups](health-checks.md)\.

1. \(Optional\) Expand **Tags** and add tags\.

1. Choose **Next**\.

1. For **Register targets**, add one or more targets as follows:
   + If the target type is **Instances**, select one or more instances, enter one or more ports, and then choose **Include as pending below**\.
   + If the target type is **IP addresses**, select the network, enter the IP address and ports, and then choose **Include as pending below**\.

1. Choose **Create target group**\.

## Step 2: Configure the load balancer and listener<a name="configure-load-balancer"></a>

Use the following procedure to create your Gateway Load Balancer\. Provide basic configuration information for your load balancer, such as a name and IP address type \(currently only IPv4 is supported\)\. Then provide information about your network, and the IP listener that routes traffic to your target groups\. Only target groups with GENEVE are available for use with the Gateway Load Balancer\.

**To create a Gateway Load Balancer**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **Load Balancing**, choose **Load Balancers**\.

1. Choose **Create Load Balancer**\.

1. Under **Gateway Load Balancer**, choose **Create**\.

1. **Basic configuration**

   1. For **Load balancer name**, enter a name for your load balancer\. For example, **my\-glb**\. The name of your Gateway Load Balancer must be unique within your set of load balancers for the Region\. It can have a maximum of 32 characters, can contain only alphanumeric characters and hyphens, and must not begin or end with a hyphen\.

   1. For **IP address type**, you must choose **IPv4**, because your clients can only use IPv4 addresses to communicate with the load balancer\.

1. **Network mapping**

   1. For **VPC**, select the service provider VPC\.

   1. For **Mappings**, select all of the Availability Zones in which you launched security appliance instances, and the corresponding public subnets\.

1. **IP listener routing**

   1. For **Default action**, select a target group to forward traffic to\. If you don't have a default target group, create a target group first\. Only target groups with GENEVE protocol are available for use with the Gateway Load Balancer\.

1. \(Optional\) Expand **Tags** and add tags\.

1. Review your configuration, and then choose **Create load balancer**\.

## Important next steps<a name="important-next-steps"></a>

After creating your load balancer, verify that your EC2 instances have passed the initial health check\. To test your load balancer, you must create a Gateway Load Balancer endpoint and update your route table to make the Gateway Load Balancer endpoint the next hop\. These configurations are set within the Amazon VPC console\. For more information, see [Step 3: Create a Gateway Load Balancer endpoint](getting-started.md#create-endpoint) and [Step 4: Configure routing](getting-started.md#configure-routing) in the [Getting started with Gateway Load Balancers](getting-started.md) section\.
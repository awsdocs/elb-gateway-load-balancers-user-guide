# Create a target group for your Gateway Load Balancer<a name="create-target-group"></a>

You register targets for your Gateway Load Balancer using a target group\.

To route traffic to the targets in a target group, create a listener and specify the target group in the default action for the listener\. For more information, see [Listeners](gateway-listeners.md)\.

You can add or remove targets from your target group at any time\. For more information, see [Register targets](target-group-register-targets.md)\. You can also modify the health check settings for your target group\. For more information, see [Modify health check settings](health-checks.md#modify-health-check-settings)\.

------
#### [ New console ]

**To create a target group using the new console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Choose **Create target group**\.

1. For **Choose a target type**, select **Instances** to register targets by instance ID or **IP addresses** to register targets by IP address\.

1. For **Target group name**, enter a name for the target group\. This name must be unique per Region per account, can have a maximum of 32 characters, must contain only alphanumeric characters or hyphens, and must not begin or end with a hyphen\.

1. For **Protocol**, use **GENEVE**\. With the GENEVE protocol, **Port** must be 6081\.

1. For **VPC**, select a virtual private cloud \(VPC\)\.

1. \(Optional\) In the **Health checks** section, modify the default settings as needed\.

1. \(Optional\) Expand the **Tags** section and add one or more tags\. To add a tag, choose **Add tag** and enter the tag key and tag value\.

1. Choose **Next**\.

1. \(Optional\) Add one or more targets as follows:
   + If the target type is **Instances**, select one or more instances, enter one or more ports, and then choose **Include as pending below**\.
   + If the target type is **IP addresses**, select the network, enter the IP address and ports, and then choose **Include as pending below**\.

1. Choose **Create target group**\.

------
#### [ Old console ]

**To create a target group using the old console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Choose **Create target group**\.

1. For **Target group name**, enter a name for the target group\. This name must be unique per Region per account, can have a maximum of 32 characters, must contain only alphanumeric characters or hyphens, and must not begin or end with a hyphen\.

1. For **Protocol**, use **GENEVE**\. With the GENEVE protocol, **Port** must be 6081\.

1. For **Target type**, select `instance` to specify targets by instance ID or `ip` to specify targets by IP address\.

1. For **VPC**, select a virtual private cloud \(VPC\)\.

1. \(Optional\) For **Health check settings** and **Advanced health check settings**, modify the default settings as needed\. Choose **Create**\.

1. \(Optional\) Add one or more tags as follows:

   1. Select the newly created target group\.

   1. Choose **Tags**, **Add/Edit Tags**\.

   1. On the **Add/Edit Tags** page, for each tag that you add, choose **Create Tag** and then specify the tag key and tag value\. When you have finished adding tags, choose **Save**\.

1. \(Optional\) To add targets to the target group, see [Register targets with your target group](target-group-register-targets.md)\.

------

**To create a target group using the AWS CLI**  
Use the [create\-target\-group](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-target-group.html) command to create the target group, the [add\-tags](https://docs.aws.amazon.com/cli/latest/reference/elbv2/add-tags.html) command to tag your target group, and the [register\-targets](https://docs.aws.amazon.com/cli/latest/reference/elbv2/register-targets.html) command to add targets\.
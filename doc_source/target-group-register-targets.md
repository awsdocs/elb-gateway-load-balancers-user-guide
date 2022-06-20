# Register targets with your target group<a name="target-group-register-targets"></a>

When your target is ready to handle requests, you register it with one or more target groups\. You can register targets by instance ID or by IP address\. The Gateway Load Balancer starts routing requests to the target as soon as the registration process completes and the target passes the initial health checks\. It can take a few minutes for the registration process to complete and health checks to start\. For more information, see [Health checks for your target groups](health-checks.md)\.

If demand on your currently registered targets increases, you can register additional targets in order to handle the demand\. If demand on your registered targets decreases, you can deregister targets from your target group\. It can take a few minutes for the deregistration process to complete and for the Gateway Load Balancer to stop routing requests to the target\. If demand increases subsequently, you can register targets that you deregistered with the target group again\. If you need to service a target, you can deregister it and then register it again when servicing is complete\.

When you deregister a target, Elastic Load Balancing waits until in\-flight requests have completed\. This is known as *connection draining*\. The status of a target is `draining` while connection draining is in progress\. After deregistration is complete, status of the target changes to `unused`\. For more information, see [Deregistration delay](target-groups.md#deregistration-delay)\.

## Target security groups<a name="target-security-groups"></a>

When you register EC2 instances as targets, you must ensure that the security groups for these instances allow inbound and outbound traffic on port 6081\.

Gateway Load Balancers do not have associated security groups\. Therefore, the security groups for your targets must use IP addresses to allow traffic from the load balancer\.

## Network ACLs<a name="network-acls"></a>

When you register EC2 instances as targets, you must ensure that the network access control lists \(ACL\) for the subnets for your instances allow traffic on port 6081\. The default network ACL for a VPC allows all inbound and outbound traffic\. If you create custom network ACLs, verify that they allow the appropriate traffic\.

## Register or deregister targets<a name="register-deregister-targets"></a>

Each target group must have at least one registered target in each Availability Zone that is enabled for the Gateway Load Balancer\.

The target type of your target group determines how you register targets with that target group\. For more information, see [Target type](target-groups.md#target-type)\.

**Requirements**
+ You cannot register instances by instance ID if they are in a VPC that is peering to the load balancer VPC \(same Region or different Region\)\. You can register these instances by IP address\.

**Topics**
+ [Register or deregister targets by instance ID](#register-instances)
+ [Register or deregister targets by IP address](#register-ip-addresses)
+ [Register or deregister targets using the AWS CLI](#register-cli)

### Register or deregister targets by instance ID<a name="register-instances"></a>

An instance must be in the `running` state when you register it\.

------
#### [ New console ]

**To register or deregister targets by instance ID using the new console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Choose the name of the target group to open its details page\.

1. Choose the **Targets** tab\.

1. To register instances, choose **Register targets**\. Select one or more instances, and then choose **Include as pending below**\. When you are finished adding instances, choose **Register pending targets**\.

1. To deregister instances, select the instance and then choose **Deregister**\.

------
#### [ Old console ]

**To register or deregister targets by instance ID using the old console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select the target group\.

1. Choose **Targets**, **Edit**\.

1. \(Optional\) For **Registered instances**, select any instances to be deregistered and choose **Remove**\.

1. \(Optional\) For **Instances**, select any running instances to be registered and then choose **Add to registered**\.

1. Choose **Save**\.

------

### Register or deregister targets by IP address<a name="register-ip-addresses"></a>

An IP address that you register must be from one of the following CIDR blocks:
+ The subnets of the VPC for the target group
+ 10\.0\.0\.0/8 \(RFC 1918\)
+ 100\.64\.0\.0/10 \(RFC 6598\)
+ 172\.16\.0\.0/12 \(RFC 1918\)
+ 192\.168\.0\.0/16 \(RFC 1918\)

------
#### [ New console ]

**To register or deregister targets by IP address using the new console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Chose the name of the target group to open its details page\.

1. Choose the **Targets** tab\.

1. To register IP addresses, choose **Register targets**\. For each IP address, select the network, Availability Zone, IP address, and port, and then choose **Include as pending below**\. When you are finished specifying addresses, choose **Register pending targets**\.

1. To deregister IP addresses, select the IP addresses and then choose **Deregister**\. If you have many registered IP addresses, you might find it helpful to add a filter or change the sort order\.

------
#### [ Old console ]

**To register or deregister targets by IP address using the old console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select the target group and choose **Targets**, **Edit**\.

1. To register IP addresses, choose the **Register targets** icon \(the plus sign\) in the menu bar\. For each IP address, specify the network, Availability Zone, IP address, and port, and then choose **Add to list**\. When you are finished specifying addresses, choose **Register**\.

1. To deregister IP addresses, choose the **Deregister targets** icon \(the minus sign\) in the menu bar\. If you have many registered IP addresses, you might find it helpful to add a filter or change the sort order\. Select the IP addresses and choose **Deregister**\.

1. To leave this screen, choose the **Back to target group** icon \(the back button\) in the menu bar\.

------

### Register or deregister targets using the AWS CLI<a name="register-cli"></a>

Use the [register\-targets](https://docs.aws.amazon.com/cli/latest/reference/elbv2/register-targets.html) command to add targets and the [deregister\-targets](https://docs.aws.amazon.com/cli/latest/reference/elbv2/deregister-targets.html) command to remove targets\.
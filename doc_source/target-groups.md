# Target groups for your Gateway Load Balancers<a name="target-groups"></a>

Each *target group* is used to route requests to one or more registered targets\. When you create a listener, you specify a target group for its default action\. Traffic is forwarded to the target group that's specified in the listener rule\. You can create different target groups for different types of requests\.

You define health check settings for your Gateway Load Balancer on a per target group basis\. Each target group uses the default health check settings, unless you override them when you create the target group or modify them later on\. After you specify a target group in a rule for a listener, the Gateway Load Balancer continually monitors the health of all targets registered with the target group that are in an Availability Zone enabled for the Gateway Load Balancer\. The Gateway Load Balancer routes requests to the registered targets that are healthy\. For more information, see [Health checks for your target groups](health-checks.md)\.

**Topics**
+ [Routing configuration](#target-group-routing-configuration)
+ [Target type](#target-type)
+ [Registered targets](#registered-targets)
+ [Target group attributes](#target-group-attributes)
+ [Deregistration delay](#deregistration-delay)
+ [Create a target group](create-target-group.md)
+ [Configure health checks](health-checks.md)
+ [Register targets](target-group-register-targets.md)
+ [Update tags](target-group-tags.md)
+ [Delete a target group](delete-target-group.md)

## Routing configuration<a name="target-group-routing-configuration"></a>

Target groups for Gateway Load Balancers support the following protocol and port:
+ **Protocol**: GENEVE
+ **Port**: 6081

## Target type<a name="target-type"></a>

When you create a target group, you specify its target type, which determines how you specify its targets\. After you create a target group, you cannot change its target type\.

The following are the possible target types:

`instance`  
The targets are specified by instance ID\.

`ip`  
The targets are specified by IP address\.

When the target type is `ip`, you can specify IP addresses from one of the following CIDR blocks:
+ The subnets of the VPC for the target group
+ 10\.0\.0\.0/8 \([RFC 1918](https://tools.ietf.org/html/rfc1918)\)
+ 100\.64\.0\.0/10 \([RFC 6598](https://tools.ietf.org/html/rfc6598)\)
+ 172\.16\.0\.0/12 \(RFC 1918\)
+ 192\.168\.0\.0/16 \(RFC 1918\)

**Important**  
You can't specify publicly routable IP addresses\.

## Registered targets<a name="registered-targets"></a>

Your Gateway Load Balancer serves as a single point of contact for clients, and distributes incoming traffic across its healthy registered targets\. Each target group must have at least one registered target in each Availability Zone that is enabled for the Gateway Load Balancer\. You can register each target with one or more target groups\.

If demand increases, you can register additional targets with one or more target groups in order to handle the demand\. The Gateway Load Balancer starts routing traffic to a newly registered target as soon as the registration process completes\.

If demand decreases, or you need to service your targets, you can deregister targets from your target groups\. Deregistering a target removes it from your target group, but does not affect the target otherwise\. The Gateway Load Balancer stops routing traffic to a target as soon as it is deregistered\. The target enters the `draining` state until in\-flight requests have completed\. You can register the target with the target group again when you are ready for it to resume receiving traffic\.

## Target group attributes<a name="target-group-attributes"></a>

The following are the target group attributes:

`deregistration_delay.timeout_seconds`  
The amount of time for Elastic Load Balancing to wait before changing the state of a deregistering target from `draining` to `unused`\. The range is 0\-3600 seconds\. The default value is 300 seconds\.

## Deregistration delay<a name="deregistration-delay"></a>

When you deregister a target, the Gateway Load Balancer manages flows to that target in the following manner: 

**New flows**:  
The Gateway Load Balancer stops sending new flows to a deregistered target\.

**Existing flows**:  
The Gateway Load Balancer handles existing flows based on protocol\.  
+  **TCP protocols**: Existing flows for TCP protocols are closed if idle for more than 350 seconds\.
+  **Non\-TCP protocols**: Existing flows for all non\-TCP protocols are closed if idle for more than 120 seconds\. 

To help drain existing flows, we recommend that you stop sending all traffic to the load balancer\. This allows the idle timeout created by deregistration to take effect\. A deregistered target shows that it is `draining` until the timeout expires\. After the deregistration delay timeout expires, the target transitions to an `unused` state\.

------
#### [ New console ]

**To update the deregistration delay value using the new console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Choose the name of the target group to open its details page\.

1. On the **Group details** page, in the **Attributes** section, choose **Edit**\.

1. On the **Edit attributes** page, change the value of **Deregistration delay** as needed\.

1. Choose **Save changes**\.

------
#### [ Old console ]

**To update the deregistration delay value using the old console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select the target group\.

1. Choose **Description**, **Edit attributes**\.

1. Change the value of **Deregistration delay** as needed, and then choose **Save**\.

------

**To update the deregistration delay value using the AWS CLI**  
Use the [modify\-target\-group\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-target-group-attributes.html) command\.
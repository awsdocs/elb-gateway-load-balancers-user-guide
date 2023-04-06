# Target groups for your Gateway Load Balancers<a name="target-groups"></a>

Each *target group* is used to route requests to one or more registered targets\. When you create a listener, you specify a target group for its default action\. Traffic is forwarded to the target group that's specified in the listener rule\. You can create different target groups for different types of requests\.

You define health check settings for your Gateway Load Balancer on a per target group basis\. Each target group uses the default health check settings, unless you override them when you create the target group or modify them later on\. After you specify a target group in a rule for a listener, the Gateway Load Balancer continually monitors the health of all targets registered with the target group that are in an Availability Zone enabled for the Gateway Load Balancer\. The Gateway Load Balancer routes requests to the registered targets that are healthy\. For more information, see [Health checks for your target groups](health-checks.md)\.

**Topics**
+ [Routing configuration](#target-group-routing-configuration)
+ [Target type](#target-type)
+ [Registered targets](#registered-targets)
+ [Target group attributes](#target-group-attributes)
+ [Deregistration delay](#deregistration-delay)
+ [Target failover](#target-failover)
+ [Flow stickiness](#flow-stickiness)
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

You can use the following attributes with target groups:

`deregistration_delay.timeout_seconds`  
The amount of time for Elastic Load Balancing to wait before changing the state of a deregistering target from `draining` to `unused`\. The range is 0\-3600 seconds\. The default value is 300 seconds\.

`stickiness.enabled`  
Indicates whether configurable flow stickiness is enabled for the target group\. The possible values are `true` or `false`\. The default is false\. When the attribute is set to `false`, 5\_tuple is used\. 

`stickiness.type`  
Indicates the type of the flow stickiness\. The possible values for target groups associated to Gateway Load Balancers are:   
+ `source_ip_dest_ip`
+ `source_ip_dest_ip_proto`

`target_failover.on_deregistration`  
Indicates how the Gateway Load Balancer handles existing flows when a target is deregistered\. The possible values are `rebalance` and `no_rebalance`\. The default is `no_rebalance`\. The two attributes \(`target_failover.on_deregistration` and `target_failover.on_unhealthy`\) can't be set independently\. The value you set for both attributes must be the same\. 

`target_failover.on_unhealthy`  
Indicates how the Gateway Load Balancer handles existing flows when a target is unhealthy\. The possible values are `rebalance` and `no_rebalance`\. The default is `no_rebalance`\. The two attributes \(`target_failover.on_deregistration` and `target_failover.on_unhealthy`\) cannot be set independently\. The value you set for both attributes must be the same\. 

## Deregistration delay<a name="deregistration-delay"></a>

When you deregister a target, the Gateway Load Balancer manages flows to that target in the following manner: 

**New flows**:  
The Gateway Load Balancer stops sending new flows to a deregistered target\.

**Existing flows**:  
The Gateway Load Balancer handles existing flows based on protocol\.  
+  **TCP protocols**: Existing flows for TCP protocols are closed if idle for more than 350 seconds\.
+  **Non\-TCP protocols**: Existing flows for all non\-TCP protocols are closed if idle for more than 120 seconds\. 

To help drain existing flows, we recommend that you stop sending all traffic to the load balancer\. This allows the idle timeout created by deregistration to take effect\. A deregistered target shows that it is `draining` until the timeout expires\. After the deregistration delay timeout expires, the target transitions to an `unused` state\.

**To update the deregistration delay value using the new console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Load Balancing**, choose **Target Groups**\.

1. Choose the name of the target group to open its details page\.

1. On the **Group details** page, in the **Attributes** section, choose **Edit**\.

1. On the **Edit attributes** page, change the value of **Deregistration delay** as needed\.

1. Choose **Save changes**\.

**To update the deregistration delay value using the AWS CLI**  
Use the [modify\-target\-group\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-target-group-attributes.html) command\.

## Target failover<a name="target-failover"></a>

With target failover, you specify how the Gateway Load Balancer handles existing traffic flows after a target becomes unhealthy or when the target is deregistered\. By default, the Gateway Load Balancer continues to send existing flows to the same target, even if the target has failed or is deregistered\. You can manage these flows by either rehashing them \(`rebalance`\) or leaving them at the default state \(`no_rebalance`\)\. 

**No rebalance**:  
The Gateway Load Balancer continues to send existing flows to failed or drained targets\. However, new flows are sent to healthy targets\. This is the default behavior\.

**Rebalance**:  
The Gateway Load Balancer rehashes existing flows and sends them to healthy targets after the deregistration delay timeout\.   
For deregistered targets, the minimum time to failover will depend on the deregistration delay\. The target is not marked as deregistered until deregistration delay is completed\.  
For unhealthy targets, the minimum time to failover will depend on the target group health check configuration \(interval times threshold\)\. This is the minimum time before which a target is flagged as unhealthy\. After this time, the Gateway Load Balancer can take several minutes due to additional propagation time and TCP retransmission backoff before it reroutes new flows to healthy targets\. 

**To update the target failover value using the new console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Choose the name of the target group to open its details page\.

1. On the **Group details** page, in the **Attributes** section, choose **Edit**\.

1. On the **Edit attributes** page, change the value of **Target failover** as needed\.

1. Choose **Save changes**\.

**To update the target failover value using the AWS CLI**  
Use the [modify\-target\-group\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-target-group-attributes.html) command, with the following key value pairs: 
+ Key=`target_failover.on_deregistration` and Value= `no_rebalance` \(default\) or `rebalance`
+ Key=`target_failover.on_unhealthy` and Value= `no_rebalance` \(default\) or `rebalance`

**Note**  
Both attributes \(`target_failover.on_deregistration` and `target_failover.on_unhealthy`\) must have the same value\. 

## Flow stickiness<a name="flow-stickiness"></a>

By default, the Gateway Load Balancer maintains stickiness of flows to a specific target appliance using 5\-tuple \(for TCP/UDP flows\)\. 5\-tuple includes source IP, source port, destination IP, destination port, and transport protocol\. You can use the stickiness type attribute to modify the default \(5\-tuple\) and choose either 3\-tuple \(source IP, destination IP, and transport protocol\) or 2\-tuple \(source IP and destination IP\)\.

**Flow stickiness considerations**
+ Flow stickiness is configured and applied at the target group level, and it applies to all traffic that goes to the target group\.
+ Flow stickiness won't work if the Gateway Load Balancer is integrated with AWS Transit Gateway when appliance mode is enabled\.
+ Flow stickiness can lead to uneven distribution of connections and flows, which can impact the availability of the target\. It is recommended that you terminate or drain all existing flows before modifying the stickiness type of the target group\.

**To update flow stickiness using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Load Balancing**, choose **Target Groups**\.

1. Choose the name of the target group to open its details page\.

1. On the **Group details** page, in the **Attributes** section, choose **Edit**\.

1. On the **Edit attributes** page, change the value of **Flow stickiness** as needed\.

1. Choose **Save changes**\.

**To enable or modify flow stickiness using the AWS CLI**  
Use the [modify\-target\-group\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-target-group-attributes.html) command with the `stickiness.enabled` and `stickiness.type` target group attributes\.
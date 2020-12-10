# Health checks for your target groups<a name="health-checks"></a>

You register your targets with one or more target groups\. Your Gateway Load Balancer starts routing requests to a newly registered target as soon as the registration process completes\. It can take a few minutes for the registration process to complete and for health checks to start\.

The Gateway Load Balancer periodically sends a request to each registered target to check its status\. Each node checks the health of each target, using the health check settings for the target group with which the target is registered\. After each health check is completed, the node closes the connection that was established for the health check\.

## Health check settings<a name="health-check-settings"></a>

You configure active health checks for the targets in a target group by using the following settings\. If the health checks exceed the specified number of **UnhealthyThresholdCount** consecutive failures, the Gateway Load Balancer takes the target out of service\. When the health checks exceed the specified number of **HealthyThresholdCount** consecutive successes, the Gateway Load Balancer puts the target back in service\.


| Setting | Description | 
| --- | --- | 
| **HealthCheckProtocol** |  The protocol that the load balancer uses when performing health checks on targets\. The possible protocols are HTTP, HTTPS, and TCP\. The default is TCP\.  | 
| **HealthCheckPort** |  The port that Gateway Load Balancer uses when performing health checks on targets\. The range is 1 to 65535\. The default is 80\.  | 
| **HealthCheckPath** |  \[HTTP/HTTPS health checks\] The ping path that is the destination on the targets for health checks\. The default is /\.  | 
| **HealthCheckTimeoutSeconds** |  The amount of time, in seconds, during which no response from a target means a failed health check\. The range is 2 to 120\. The default is 5\.  | 
| **HealthCheckIntervalSeconds** |  The approximate amount of time, in seconds, between health checks of an individual target\. The range is 5 to 300\. The default is 10 seconds\. This value must be greater than or equal to **HealthCheckTimeoutSeconds**\.  | 
| **HealthyThresholdCount** |  The number of consecutive successful health checks required before considering an unhealthy target healthy\. The range is 2 to 10\. The default is 3\.  | 
| **UnhealthyThresholdCount** |  The number of consecutive failed health checks required before considering a target unhealthy\. The range is 2 to 10\. The default is 3\.  | 
| **Matcher** |  \[HTTP/HTTPS health checks\] The HTTP codes to use when checking for a successful response from a target\. This value must be 200\-399\.  | 

## Target health status<a name="target-health-states"></a>

Before the Gateway Load Balancer sends a health check request to a target, you must register it with a target group, specify its target group in a listener rule, and ensure that the Availability Zone of the target is enabled for the Gateway Load Balancer\.

The following table describes the possible values for the health status of a registered target\.


| Value | Description | 
| --- | --- | 
| `initial` |  The Gateway Load Balancer is in the process of registering the target or performing the initial health checks on the target\. Related reason codes: `Elb.RegistrationInProgress` \| `Elb.InitialHealthChecking`  | 
| `healthy` |  The target is healthy\. Related reason codes: None  | 
| `unhealthy` |  The target did not respond to a health check or failed the health check\. Related reason code: `Target.FailedHealthChecks`  | 
| `unused` |  The target is not registered with a target group, the target group is not used in a listener rule, the target is in an Availability Zone that is not enabled, or the target is in the stopped or terminated state\. Related reason codes: `Target.NotRegistered` \| `Target.NotInUse` \| `Target.InvalidState` \| `Target.IpUnusable`  | 
| `draining` |  The target is deregistering and connection draining is in process\. Related reason code: `Target.DeregistrationInProgress`  | 
| `unavailable` |  Target health is unavailable\. Related reason code: `Elb.InternalError`  | 

## Health check reason codes<a name="target-health-reason-codes"></a>

If the status of a target is any value other than `Healthy`, the API returns a reason code and a description of the issue, and the console displays the same description\. Reason codes that begin with `Elb` originate on the Gateway Load Balancer side and reason codes that begin with `Target` originate on the target side\.


| Reason code | Description | 
| --- | --- | 
| `Elb.InitialHealthChecking` |  Initial health checks in progress  | 
| `Elb.InternalError` |  Health checks failed due to an internal error  | 
| `Elb.RegistrationInProgress` |  Target registration is in progress  | 
| `Target.DeregistrationInProgress` |  Target deregistration is in progress  | 
| `Target.FailedHealthChecks` |  Health checks failed  | 
| `Target.InvalidState` |  Target is in the stopped state Target is in the terminated state Target is in the terminated or stopped state Target is in an invalid state  | 
| `Target.IpUnusable` |  The IP address cannot be used as a target, as it is in use by a load balancer  | 
| `Target.NotInUse` |  Target group is not configured to receive traffic from the Gateway Load Balancer Target is in an Availability Zone that is not enabled for the Gateway Load Balancer  | 
| `Target.NotRegistered` |  Target is not registered to the target group  | 

## Check the health of your targets<a name="check-target-health"></a>

You can check the health status of the targets registered with your target groups\.

------
#### [ New console ]

**To check the health of your targets using the new console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Choose the name of the target group to open its details page\.

1. On the **Targets** tab, the **Status** column indicates the status of each target\.

1. If the target status is any value other than `Healthy`, the **Status details** column contains more information\.

------
#### [ Old console ]

**To check the health of your targets using the old console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select the target group\.

1. Choose **Targets**, and view the status of each target in the **Status** column\. If the status is any value other than `Healthy`, the console displays more information\.

------

**To check the health of your targets using the AWS CLI**  
Use the [describe\-target\-health](https://docs.aws.amazon.com/cli/latest/reference/elbv2/describe-target-health.html) command\. The output of this command contains the target health state\. It includes a reason code if the status is any value other than `Healthy`\.

**To receive email notifications about unhealthy targets**  
Use CloudWatch alarms to trigger a Lambda function to send details about unhealthy targets\. For step\-by\-step instructions, see the following blog post: [Identifying unhealthy targets of your load balancer](http://aws.amazon.com/blogs/networking-and-content-delivery/identifying-unhealthy-targets-of-elastic-load-balancer/)\.

## Modify health check settings<a name="modify-health-check-settings"></a>

You can modify some of the health check settings for your target group\.

------
#### [ New console ]

**To modify health check settings for a target group using the new console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Choose the name of the target group to open its details page\.

1. On the **Group details** tab, in the **Health check settings** section, choose **Edit**\.

1. On the **Edit health check settings** page, modify the settings as needed, and then choose **Save changes**\.

------
#### [ Old console ]

**To modify health check settings for a target group using the old console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select the target group\.

1. Choose **Health checks**, **Edit**\.

1. On the **Edit target group** page, modify the settings as needed, and then choose **Save**\.

------

**To modify health check settings for a target group using the AWS CLI**  
Use the [modify\-target\-group](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-target-group.html) command\.
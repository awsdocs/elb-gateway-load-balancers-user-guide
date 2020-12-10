# Listeners for your Gateway Load Balancers<a name="gateway-listeners"></a>

When you create your Gateway Load Balancer, you add a *listener*\. A listener is a process that checks for connection requests\.

Listeners for Gateway Load Balancers listen for all IP packets across all ports\. You cannot specify a protocol or port when you create a listener for a Gateway Load Balancer\. You cannot delete the listener for a Gateway Load Balancer\.

When you create a listener, you specify a rule for routing requests\. This rule forwards requests to the specified target group\. You can update the listener rule to forward requests to a different target group\.

**To update your listener using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer and choose **Listeners**\.

1. Choose **Edit listener**\.

1. For **Forwarding to target group**, choose a target group\.

1. Choose **Save**\.

**To update your listener using the AWS CLI**  
Use the [modify\-listener](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-listener.html) command\.
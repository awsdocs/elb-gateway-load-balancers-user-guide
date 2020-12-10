# Delete a Gateway Load Balancer<a name="delete-load-balancer"></a>

As soon as your Gateway Load Balancer becomes available, you are billed for each hour or partial hour that you keep it running\. When you no longer need the Gateway Load Balancer, you can delete it\. As soon as the Gateway Load Balancer is deleted, you stop incurring charges for it\.

You can't delete a Gateway Load Balancer if it is in use by another service\. For example, if the Gateway Load Balancer is associated with a VPC endpoint service, you must delete the endpoint service configuration before you can delete the associated Gateway Load Balancer\.

Deleting a Gateway Load Balancer also deletes its listeners\. Deleting a Gateway Load Balancer does not affect its registered targets\. For example, your EC2 instances continue to run and are still registered to their target groups\. To delete your target groups, see [Delete a target group](delete-target-group.md)\.

**To delete n Gateway Load Balancer using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the Gateway Load Balancer\.

1. Choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Yes, Delete**\.

**To delete a Gateway Load Balancer using the AWS CLI**  
Use the [delete\-load\-balancer](https://docs.aws.amazon.com/cli/latest/reference/elbv2/delete-load-balancer.html) command\.
# Gateway Load Balancers<a name="gateway-load-balancers"></a>

Use a Gateway Load Balancer to deploy and manage a fleet of virtual appliances that support the GENEVE protocol\.

A Gateway Load Balancer operates at the third layer of the Open Systems Interconnection \(OSI\) model\. It listens for all IP packets across all ports and forwards traffic to the target group that's specified in the listener rule, using the GENEVE protocol on port 6081\.

You can add or remove targets from your load balancer as your needs change, without disrupting the overall flow of requests\. Elastic Load Balancing scales your load balancer as traffic to your application changes over time\. Elastic Load Balancing can scale to the vast majority of workloads automatically\.

**Topics**
+ [Load balancer state](#load-balancer-state)
+ [IP address type](#ip-address-type)
+ [Load balancer attributes](#load-balancer-attributes)
+ [Availability Zones](#availability-zones)
+ [Network maximum transmission unit \(MTU\)](#mtu)
+ [Deletion protection](#deletion-protection)
+ [Cross\-zone load balancing](#cross-zone-load-balancing)
+ [Asymmetric flows](#asymmetric-flows)
+ [Create a load balancer](create-load-balancer.md)
+ [Update tags](tag-load-balancer.md)
+ [Delete a load balancer](delete-load-balancer.md)

## Load balancer state<a name="load-balancer-state"></a>

A Gateway Load Balancer can be in one of the following states:

`provisioning`  
The Gateway Load Balancer is being set up\.

`active`  
The Gateway Load Balancer is fully set up and ready to route traffic\.

`failed`  
The Gateway Load Balancer could not be set up\.

## IP address type<a name="ip-address-type"></a>

You can set the types of IP addresses that the application servers can use to access your Gateway Load Balancers\. The following are the supported IP address types:
+ `ipv4` – Only IPv4 is supported\.
+ `dualstack` – Both IPv4 and IPv6 are supported\.

**Dualstack load balancer considerations**
+ The virtual private cloud \(VPC\) and subnets that you specify for the load balancer must have associated IPv6 CIDR blocks\.
+ The route tables for the subnets in the service consumer VPC must route IPv6 traffic, and the network ACLs for these subnets must allow IPv6 traffic\.
+ A Gateway Load Balancer encapsulates both IPv4 and IPv6 client traffic with an IPv4 GENEVE header and sends it to the appliance\. The appliance encapsulates both IPv4 and IPv6 client traffic with an IPv4 GENEVE header and sends it back to the Gateway Load Balancer\.

You can set the IP address type when you create the load balancer\. You can also update it at any time\.

**To update the IP address type using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer\.

1. Choose **Actions**, **Edit IP address type**\.

1. For **IP address type**, choose **ipv4** to support IPv4 addresses only or **dualstack** to support both IPv4 and IPv6 addresses\.

1. Choose **Save**\.

**To update the IP address type using the AWS CLI**  
Use the [set\-ip\-address\-type](https://docs.aws.amazon.com/cli/latest/reference/elbv2/set-ip-address-type.html) command\.

## Load balancer attributes<a name="load-balancer-attributes"></a>

The following are the load balancer attributes for Gateway Load Balancers:

`deletion_protection.enabled`  
Indicates whether [deletion protection](#deletion-protection) is enabled\. The default is `false`\.

`load_balancing.cross_zone.enabled`  
Indicates whether [cross\-zone load balancing](#cross-zone-load-balancing) is enabled\. The default is `false`\.

## Availability Zones<a name="availability-zones"></a>

When you create a Gateway Load Balancer, you enable one or more Availability Zones, and specify the subnet that corresponds to each zone\. When you enable multiple Availability Zones, it ensures that the load balancer can continue to route traffic even if an Availability Zone becomes unavailable\. The subnets that you specify must each have at least 8 available IP addresses\. Subnets cannot be added or removed after the load balancer is created\. To add or remove a subnet, you must create a new load balancer\.

## Network maximum transmission unit \(MTU\)<a name="mtu"></a>

The maximum transmission unit \(MTU\) is the size of the largest data packet that can be transmitted through the network\. The Gateway Load Balancer interface MTU supports packets up to 8,500 bytes\. Packets with a size larger than 8500 bytes that arrive at the Gateway Load Balancer interface are dropped\.

A Gateway Load Balancer encapsulates IP traffic with a GENEVE header and forwards it to the appliance\. The GENEVE encapsulation process adds 64 bytes to the original packet\. Therefore, to support packets up to 8,500 bytes, ensure that the MTU setting of your appliance supports packets of at least 8,564 bytes\.

Gateway Load Balancers do not support IP fragmentation\. Additionally, Gateway Load Balancers do not generate ICMP message "Destination Unreachable: fragmentation needed and DF set"\. Due to this, Path MTU Discovery \(PMTUD\) is not supported\.

## Deletion protection<a name="deletion-protection"></a>

To prevent your Gateway Load Balancer from being deleted accidentally, you can enable deletion protection\. By default, deletion protection is disabled\.

If you enable deletion protection for your Gateway Load Balancer, you must disable it before you can delete the Gateway Load Balancer\.

**To enable deletion protection using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the Gateway Load Balancer\.

1. Choose **Actions**, **Edit attributes**\.

1. On the **Edit load balancer attributes** page, select **Enable** for **Delete Protection**, and then choose **Save**\.

**To disable deletion protection using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the Gateway Load Balancer\.

1. Choose **Actions**, **Edit attributes**\.

1. On the **Edit load balancer attributes** page, clear **Enable** for **Delete Protection**, and then choose **Save**\.

**To enable or disable deletion protection using the AWS CLI**  
Use the [modify\-load\-balancer\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-load-balancer-attributes.html) command with the `deletion_protection.enabled` attribute\.

## Cross\-zone load balancing<a name="cross-zone-load-balancing"></a>

By default, each load balancer node distributes traffic across the registered targets in its Availability Zone only\. If you enable cross\-zone load balancing, each Gateway Load Balancer node distributes traffic across the registered targets in all enabled Availability Zones\. For more information, see [Cross\-zone load balancing](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html#cross-zone-load-balancing) in the *Elastic Load Balancing User Guide*\.

**To enable cross\-zone load balancing using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the Gateway Load Balancer\.

1. Choose **Actions**, **Edit attributes**\.

1. On the **Edit load balancer attributes** page, select **Enable** for **Cross\-Zone Load Balancing**, and then choose **Save**\.

**To enable cross\-zone load balancing using the AWS CLI**  
Use the [modify\-load\-balancer\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-load-balancer-attributes.html) command with the `load_balancing.cross_zone.enabled` attribute\.

## Asymmetric flows<a name="asymmetric-flows"></a>

Gateway Load Balancers support asymmetric flows when the load balancer processes the initial flow packet and the response flow packet is not routed through the load balancer\. Gateway Load Balancers do not support asymmetric flows when the load balancer does not process the initial flow packet but the response flow packet is routed through the load balancer\.
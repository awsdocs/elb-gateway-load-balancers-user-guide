# What is a Gateway Load Balancer?<a name="introduction"></a>

Gateway Load Balancers enable you to deploy, scale, and manage virtual appliances, such as firewalls, intrusion detection and prevention systems, and deep packet inspection systems\.

A Gateway Load Balancer operates at the third layer of the Open Systems Interconnection \(OSI\) model, the network layer\. It listens for all IP packets across all ports and forwards traffic to the target group that's specified in the listener rule\. The Gateway Load Balancer and its registered virtual appliance instances exchange application traffic using the [GENEVE](https://datatracker.ietf.org/doc/html/rfc8926) protocol on port 6081\. It supports a maximum transmission unit \(MTU\) size of 8500 bytes\.

Gateway Load Balancers use Gateway Load Balancer endpoints to securely exchange traffic across VPC boundaries\. A Gateway Load Balancer endpoint is a VPC endpoint that provides private connectivity between virtual appliances in the service provider VPC and application servers in the service consumer VPC\. You deploy the Gateway Load Balancer in the same VPC as the virtual appliances\. You register the virtual appliances with a target group for the Gateway Load Balancer\.

Traffic to and from a Gateway Load Balancer endpoint is configured using route tables\. Traffic flows from the service consumer VPC over the Gateway Load Balancer endpoint to the Gateway Load Balancer in the service provider VPC, and then returns to the service consumer VPC\. You must create the Gateway Load Balancer endpoint and the application servers in different subnets\. This enables you to configure the Gateway Load Balancer endpoint as the next hop in the route table for the application subnet\.

For more information, see [Gateway Load Balancer endpoints \(AWS PrivateLink\)](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-gateway-load-balancer.html) in the *Amazon VPC User Guide*\.

## Appliance vendors<a name="appliance-vendors"></a>

You are responsible for choosing and qualifying software from appliance vendors\. You must trust the appliance software to inspect or modify traffic from the load balancer\. The appliance vendors listed as [Elastic Load Balancing Partners](http://aws.amazon.com/elasticloadbalancing/partners/) have integrated and qualified their appliance software with AWS\. You can place a higher degree of trust in the appliance software from vendors in this list\. However, AWS does not guarantee the security or reliability of software from these vendors\.

## Getting started<a name="tutorials"></a>

To create a Gateway Load Balancer using the AWS Management Console, see [Getting started](getting-started.md)\. To create a Gateway Load Balancer using the AWS Command Line Interface, see [Getting started using the CLI](getting-started-cli.md)\.

## Pricing<a name="pricing"></a>

With your load balancer, you pay only for what you use\. For more information, see [Elastic Load Balancing pricing](http://aws.amazon.com/elasticloadbalancing/pricing/)\.

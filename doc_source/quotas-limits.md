# Quotas for your Gateway Load Balancers<a name="quotas-limits"></a>

Your AWS account has default quotas, formerly referred to as limits, for each AWS service\. Unless otherwise noted, each quota is Region\-specific\. You can request increases for some quotas, and other quotas cannot be increased\.

To request a quota increase, use the [limit increase form](https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase)

**Load balancers**  
Your AWS account has the following quotas related to Gateway Load Balancers\.


| Name | Default | Adjustable | 
| --- | --- | --- | 
| Gateway Load Balancers per Region | 100 | Yes | 
| Gateway Load Balancers per VPC | 100 | Yes | 
| Gateway Load Balancer ENIs per VPC | 300 \* | Yes | 
| Listeners per Gateway Load Balancer | 1 | No | 

**\*** Each Gateway Load Balancer uses one network interface per zone\. 

**Target groups**  
The following quotas are for target groups\.


| Name | Default | Adjustable | 
| --- | --- | --- | 
| GENEVE target groups per Region | 100 | Yes | 
| Targets per target group | 1,000 | Yes | 
| Targets per Availability Zone per GENEVE target group | 300 | No | 
| Targets per Availability Zone per Gateway Load Balancer | 300 | No | 
| Targets per Gateway Load Balancer | 300 | No | 

**Bandwidth**  
By default, each VPC endpoint can support a bandwidth of up to 10 Gbps per Availability Zone and automatically scales up to 100 Gbps\. If your application needs higher throughput, contact AWS support\.
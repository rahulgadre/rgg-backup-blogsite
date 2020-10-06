---
layout: post
title: "Configuring VPC in AWS"
date: 2020-10-6
excerpt: "Configuring VPC in AWS"
tags: [AWS]
comments: true
---

VPC stands for virtual private cloud and it allows users to create a virtual datacenter in the cloud. 

The following steps describe how to configure VPC in AWS:

- Create a VPC – 10.0.0.0/16
- Create two subnets and one in each AZ. Mention either AZ name or public/private for simplicity.
- Subnets created in this exercise: 10.0.1.0/24, 10.0.2.0/24.
- Create IGW and attach it to your VPC.
- Create a new route table and add this route 0.0.0.0/0 and attach it your IGW.
- Subnet association: add public subnet to this new route table
- Create 2 EC2 instances, 1 instance in public and 1 in private subnet
- For the public EC2 instance, create a new SG (SSH, HTTP, HTTPS).
- For the private EC2 instance, create a new SG (SSH, HTTP, HTTPS, ICMP) and add public network as source network.
- To allow Internet access for the private EC2, create a NAT instance (disable source/destination) or NAT GW and put it in the public subnet. Assign the SG used for the public instance.
- Add a route 0.0.0.0/0  in your main VPC and attach it your NAT instance or NAT GW.
- NACL: Additional layer of security. Create a new NACL for public subnet and configure inbound and outbound rules. Add the public subnet to this newly created NACL.
- NACL is stateless so inbound and outbound both rules are required. IPv4 rules start with 100 & IPv6 101.  If a rule/condition is met, an action is taken and the remaining rules are ignored.
- Don’t forget to add ephemeral ports (1024-65535) in the outbound rules as EC2 instances require these ports open to communicate.
- Clean-up: Delete NAT GW, IGW, and Elastic IP first and then delete the VPC.
- Remember: One NACL/subnet & one subnet/AZ.

Note: When a VPC is created, it also creates a route table, NACL, and security group. 

The following diagram shows all the components involved in a VPC.

![](/assets/img/vpc.png)

Image credit: acloudguru
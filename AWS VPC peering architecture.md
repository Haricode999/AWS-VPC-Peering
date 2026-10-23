# AWS VPC peering: bastion to private workloads

Two private VPCs joined by a direct peering link. One internet entry point.

AWS region: ap-south-1 (Mumbai)

VPC A: ewb-vpc-a (requester)

vpc-0a45e933c94dbbf36 | 10.10.0.0/16

**ewb-igw-a**Target: 0.0.0.0/0

**Local admin laptop**ProxyJump ewb-bastion

*SSH / 22*

Subnet: ewb-a-public 10.10.1.0/24

ewb-bastion

i-00c1142d3444dc204

Private *10.10.1.10* | Public *13.235.114.229*

SG ewb-sg-bastion: SSH TCP 22 from student IP /32

*ProxyJump*

Subnet: ewb-a-private 10.10.2.0/24

ewb-app-a

i-01b87853625974f02

Private *10.10.2.10* | Public IP: none

SSH from 10.10.1.10/32

ICMP from 10.20.1.10/32

**VPC PEERING**pcx-0b613354c332e40d5 ACTIVE No transitive routing

VPC B: ewb-vpc-b (accepter)

Zero IGW / air-gapped

vpc-0bc1f49b0c3499717 | 10.20.0.0/16

Subnet: ewb-b-private 10.20.1.0/24

Route table: rtb-0a5b202e2cb2445ad

ewb-app-b

i-030abb340260099c4

Private *10.20.1.10* | Public IP: none

SG ewb-sg-app-b (sg-0c53acb5ccaada0e7)

TCP 22 inbound from bastion 10.10.1.10/32

ICMP echo request inbound from App-A 10.10.2.10/32

Stateful return: echo reply automatically allowed

VPC A 10.10.0.0/16 VPC B 10.20.0.0/16 ICMP cross-VPC echo path OpenSSH ProxyJump control path Zero internet access to VPC B
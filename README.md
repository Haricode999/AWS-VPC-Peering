# AWS-VPC-Peering
Desingned and implemented a Secure AWS VPC Peering architecture connecting two isolated VPCs. Configured bidirectional routing, private subnets, least-privilege Security Groups, and Bastion-host SSH access using OpenSSH. Verified cross-VPC connectivity through ICMP testing 

## Architecture Diagram

![AWS VPC Peering Architecture](6260330900746867079.jpg)

## Project Overview

This lab uses two VPCs in the **same AWS account and AWS Region
(`ap-south-1`)**.

-   **VPC A** contains a public subnet with a bastion host and a private
    subnet with App-A.
-   **VPC B** contains a private subnet with App-B and has **no Internet
    Gateway**.
-   A **VPC peering connection** enables private-IP communication
    between the application workloads.
-   The laptop connects to the bastion host over SSH. From the bastion,
    SSH access to the private instances is performed using `ProxyJump`
    or a separate SSH hop.
-   VPC peering provides private connectivity only; it does **not**
    automatically give VPC B internet access through VPC A.

## Network Address Plan

  -----------------------------------------------------------------------
  Resource                CIDR / Private IP       Purpose
  ----------------------- ----------------------- -----------------------
  `ewb-vpc-a`             `10.10.0.0/16`          VPC A

  `ewb-a-public`          `10.10.1.0/24`          Public subnet
                                                  containing the bastion

  Bastion host            `10.10.1.10`            SSH entry point

  `ewb-a-private`         `10.10.2.0/24`          Private application
                                                  subnet in VPC A

  App-A                   `10.10.2.10`            Private workload in VPC
                                                  A

  `ewb-vpc-b`             `10.20.0.0/16`          VPC B

  `ewb-b-private`         `10.20.1.0/24`          Private application
                                                  subnet in VPC B

  App-B                   `10.20.1.10`            Private workload in VPC
                                                  B
  -----------------------------------------------------------------------

## Main AWS Components

-   **Amazon VPC:** Two isolated virtual networks with non-overlapping
    CIDR ranges.
-   **Subnets:** One public and one private subnet in VPC A, and one
    private subnet in VPC B.
-   **Internet Gateway:** Attached to VPC A only, to provide the
    intended internet entry path for the bastion.
-   **VPC Peering:** Private network connection between VPC A and VPC B.
-   **Route tables:** Routes for each VPC's local CIDR and explicit
    routes to the peer VPC CIDR through the peering connection.
-   **Security groups:** Restrict SSH and ICMP traffic to the intended
    sources.
-   **Bastion host:** Provides controlled administrative access to
    private instances.

    ## Routing Requirements

After the peering connection is accepted, configure routes in **both
directions**:

  Route table                 Destination      Target
  --------------------------- ---------------- ------------------------
  VPC A public route table    `10.20.0.0/16`   VPC peering connection
  VPC A private route table   `10.20.0.0/16`   VPC peering connection
  VPC B private route table   `10.10.0.0/16`   VPC peering connection

Keep the existing local routes in all route tables. The public route
table in VPC A should also have `0.0.0.0/0` pointing to VPC A's Internet
Gateway if internet access is required for the bastion.

> **Important:** A route in a route table does not by itself permit
> traffic. Security groups and network ACLs must also allow the relevant
> traffic. Do not add a default route to an Internet Gateway in VPC B
> for this lab.

## Security Design

-   Allow SSH (`TCP/22`) to the bastion only from the administrator's
    current public IPv4 address using a `/32` source.
-   Allow SSH to private instances only from the intended administrative
    source, such as the bastion host or its security group, according to
    the configured access design.
-   Allow ICMP Echo Request only from the intended peer workload when
    validating connectivity.
-   Avoid broad inbound rules such as `0.0.0.0/0` for SSH.
-   Keep private application instances without public IPv4 addresses.

Security group rules should match the exact addresses and access paths
configured in the deployed lab.

## Validation Checklist

-   [ ] Both VPCs and all subnets use the planned, non-overlapping CIDR
    ranges.
-   [ ] The Internet Gateway is attached to VPC A only.
-   [ ] The VPC peering connection is active.
-   [ ] Required peer-CIDR routes exist in the relevant route tables in
    both VPCs.
-   [ ] Security groups permit only the intended SSH and ICMP traffic.
-   [ ] The administrator can SSH to the bastion.
-   [ ] The administrator can reach the private instances using the
    configured SSH hop / `ProxyJump` workflow.
-   [ ] App-A and App-B can communicate over their private IP addresses
    when the required routes and security rules allow it.
-   [ ] VPC B does not gain internet access merely because VPC peering
    is configured.

## What This Project Demonstrates

-   AWS VPC and subnet design
-   Public vs. private subnet routing
-   Internet Gateway configuration
-   VPC peering and bidirectional routing
-   Security group least-privilege rules
-   Bastion-based SSH administration
-   Private-IP connectivity validation and troubleshooting

## Cleanup

To avoid ongoing AWS charges, terminate any EC2 instances that are no
longer needed and delete lab resources when finished. Remove
dependencies in the correct order---for example, delete the peering
connection before removing routes that target it, then clean up
instances, security groups, subnets, route tables, gateways, and VPCs as
appropriate.

# AWS Networking Web Server Lab

## Project Overview

In this project, I built a small AWS network from scratch and deployed an Nginx web server on an EC2 instance.

My main goal was not only to make the website work, but to understand how the network traffic moves through AWS and how to troubleshoot the environment when something breaks.

I manually created the VPC, subnet, route table, Internet Gateway, Security Group, EC2 instance, and Nginx web server.

After the website was working, I intentionally broke different parts of the environment and diagnosed the problems.

---

## Architecture

VPC:

10.0.0.0/16

Public Subnet:

10.0.1.0/24

Architecture:

Internet
|
v
Internet Gateway
|
v
Public Route Table
|
v
Public Subnet
|
v
Security Group
|
v
EC2 Instance
|
v
Nginx
|
v
Website

---

## AWS Services Used

- Amazon VPC
- EC2
- Internet Gateway
- Route Tables
- Security Groups
- Network ACL
- Public and Private IPv4 addressing

---

## Linux Tools Used

- SSH
- systemctl
- ss
- Nginx
- nc

---

## Network Configuration

### VPC

10.0.0.0/16

This gives the VPC a large private IPv4 address space.

### Public Subnet

10.0.1.0/24

A /24 subnet contains 256 total IPv4 addresses.

AWS reserves five addresses in an IPv4 subnet, leaving 251 usable addresses.

### Route Table

The public route table contained:

10.0.0.0/16 -> local

0.0.0.0/0 -> Internet Gateway

The local route allows communication inside the VPC.

The 0.0.0.0/0 route sends internet-bound traffic to the Internet Gateway.

---

## EC2 Web Server

I launched an Amazon Linux EC2 instance inside the public subnet.

The instance had:

- Private IPv4 address
- Public IPv4 address
- SSH access from my IP only
- HTTP port 80 open to the internet

I connected to the server using SSH.

Example:

ssh -i "key.pem" ec2-user@PUBLIC-IP

I installed Nginx and started the service.

sudo dnf install nginx -y

sudo systemctl start nginx

sudo systemctl enable nginx

I confirmed Nginx was running with:

sudo systemctl status nginx

I checked the listening ports with:

sudo ss -tulpn | grep :80

Nginx was listening on port 80.

---

# Troubleshooting Labs

## Problem 1 - Wrong Route Table Association

When I created a replacement subnet, the subnet was not associated with my public route table.

Because of this, SSH connections to the EC2 instance timed out.

I tested port 22 using:

nc -vz PUBLIC-IP 22

The connection timed out.

I discovered that the new subnet was not associated with the route table containing:

0.0.0.0/0 -> Internet Gateway

After associating the correct route table with the subnet, SSH started working.

### Lesson Learned

Having an Internet Gateway attached to a VPC is not enough.

The subnet must use a route table that has a route to the Internet Gateway.

---

## Problem 2 - Security Group Blocking HTTP

I removed the HTTP port 80 inbound rule from the EC2 Security Group.

The route table and Nginx were still working, but the website stopped loading.

I restored:

HTTP
TCP
Port 80
Source 0.0.0.0/0

The website started working again.

### Lesson Learned

A working route does not automatically mean traffic is allowed.

The Security Group must also allow the required port.

---

## Problem 3 - Missing Internet Gateway Route

I removed:

0.0.0.0/0 -> Internet Gateway

from the public subnet route table.

The EC2 still had a public IP and Nginx was still running, but the website could no longer be reached.

After restoring the route, the website worked again.

### Lesson Learned

A public IP alone does not give an EC2 instance internet connectivity.

The subnet also needs a correct route to the Internet Gateway.

---

## Problem 4 - Nginx Service Stopped

I stopped Nginx:

sudo systemctl stop nginx

AWS networking was still configured correctly, but the website stopped working.

This showed that a website failure is not always caused by AWS networking.

I checked the service using:

sudo systemctl status nginx

Then restarted it:

sudo systemctl start nginx

### Lesson Learned

Routing and firewall rules can be correct while the application itself is down.

---

## Problem 5 - Wrong Application Port

I changed Nginx from port 80 to port 8080.

Nginx was running, but the Security Group was still allowing only port 80.

This caused a mismatch between the application listening port and the allowed network port.

I checked the listening port using:

sudo ss -tulpn

### Lesson Learned

The application listening port and the firewall rules must match.

---

# Main Things I Learned

This project helped me understand the difference between:

- Routing and security
- Public and private IP addresses
- VPCs and subnets
- Internet Gateways and route tables
- Security Groups and NACLs
- Network failures and application failures
- Listening ports and allowed ports

Instead of guessing when something stopped working, I learned to troubleshoot the network layer by layer.

---

## Next Project

My next project will expand this architecture using:

- Private subnets
- NAT Gateway
- Application Load Balancer
- Target Groups
- Multiple Availability Zones
- Auto Scaling
- CloudWatch

The goal will be to build a more secure and highly available AWS architecture.

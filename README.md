# Highly Available WordPress Application

## Project Overview

This repository documents the implementation of a highly available 3-tier architecture for a WordPress web application on Amazon Web Services (AWS). The project ensures a robust network, a high availability data tier using Amazon RDS and ElastiCache, and a scalable application tier with load balancing and auto-scaling capabilities.

## Architecture

### Network Setup

- **VPC Creation:** A Virtual Private Cloud (VPC) named "Wordpress-VPC" was created.
- **Subnet Configuration:** Established 6 subnets across two availability zones: 2 Public, 2 Application, and 2 Data.
- **Internet Gateway:** Attached an Internet Gateway to the Wordpress VPC.
- **Gateways and Routing:** Created two Gateways (one for each public subnet A and B), configured route tables for Application subnets using NAT gateways as the default gateway, and associated the route table with both Application subnets.

### High Availability Data Tier

#### Amazon RDS

- **Security Groups:** Two security groups were created — one for the client database and another for the main database.
- **Inbound Rules:** Edited inbound rules for the main database to allow inbound traffic from the client database security group.
- **RDS Subnet Group:** Formed an RDS subnet group to specify subnets for database deployment.
- **Aurora Database Cluster:** Created an Aurora database cluster connected to the VPC and DB subnet group.

#### Amazon ElastiCache

- **Cache Security Groups:** Established cache security groups for the cache client and the main cache.
- **Inbound Traffic Rules:** Modified inbound traffic rules for the main cache to allow traffic from the cache client security group.
- **Memcached Configuration:** Configured an ElastiCache Memcached instance, adding a subnet group consisting of Data A and Data B, along with the main cache security group.

#### Amazon EFS

- **Filesystem Security Groups:** Made filesystem security groups for Elastic Filesystem (EFS).
- **Inbound Rules:** Edited inbound rules for traffic coming from FS client security group to the main FS security group.
- **EFS Cluster:** Configured an EFS cluster, mounting the two Application subnets (A and B) in each availability zone.

### Application Tier

#### Load Balancer

- **Load Balancer Security Group:** Created a load balancer security group, allowing HTTP traffic on port 80 only from a specified IP.
- **Configuration:** Configured a load balancer to distribute application traffic across multiple targets, such as EC2 instances in different AZ zones.

#### Launch Template

- **WordPress Servers Security Group:** Created a security group for WordPress servers, allowing HTTP traffic on port 80 only from the Load Balancer security group.
- **Launch Template:** Configured a launch template for Auto Scaling Groups, including a custom user data script with EFS, DB name, host, DB username, DB password, and the load balancer. Also included 4 security groups in the launch template: WP cache client, WP DB client, WP EFS client, and WP Wordpress.

#### App Server Auto Scaling Group

- **Auto Scaling Group:** Created an Auto Scaling Group to automatically scale the number of WordPress application servers based on traffic.
- **Access:** Accessed the WordPress installation site through the DNS name of the Application Load Balancer.

## Deployment

1. **Network Creation:** Execute the network creation script to establish the VPC, subnets, and gateways.
   ```bash
   sh create_network.sh
   ```

2. **Data Tier Setup:** Run the RDS and ElastiCache setup script to create the database and cache layers.
   ```bash
   sh setup_data_tier.sh
   ```

3. **Shared Filesystem Configuration:** Execute the EFS setup script for configuring the shared filesystem.
   ```bash
   sh configure_shared_filesystem.sh
   ```

4. **Application Tier Deployment:** Run the application tier deployment script to set up load balancing, launch templates, and the auto-scaling group.
   ```bash
   sh deploy_application_tier.sh
   ```

## Summary

- **AWS VPC:** A software-defined network across multiple AWS availability zones.
- **Amazon RDS:** Managed active/standby multi-node MySQL database deployment with automatic backups and recovery.
- **Amazon ElastiCache:** Memcached-powered cache to reduce database queries against the HA MySQL database.
- **Amazon EFS:** Distributed NFS share for sharing a single WordPress installation across multiple application servers.
- **AWS Auto Scaling and Application Load Balancer:** Ensures a dynamic and scalable infrastructure based on resource utilization and user traffic.

This project results in a highly-available, distributed, and fault-tolerant web application. The demonstrated pattern can be reused for various web application technologies wherever state can be externalized to a filesystem, cache, or database. Feel free to use and modify this template for your own projects.

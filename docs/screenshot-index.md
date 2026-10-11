# Screenshot index

This index maps each image to the work it documents. Open README.md for the full explanation and image embedded under each step.

## 01. VPC created

File: `screenshots/01-vpc-created.jpg`

The VPC is the isolated network for the project. The screenshot confirms the VPC exists and shows its IPv4 CIDR, 10.0.0.0/16.

## 02. Six subnets created

File: `screenshots/02-six-subnets-created.jpg`

Six /24 subnets were created: two public, two private application, and two private database subnets. The CIDR ranges are 10.0.1.0/24 through 10.0.6.0/24.

## 03. Public route table association

File: `screenshots/03-public-route-table-associated.jpg`

The public route table is associated with the two public subnets. This connects the public subnets to the routing configuration used for internet-facing resources.

## 04. Private route table association

File: `screenshots/04-private-route-table-associated.jpg`

The private application route table is associated with the two private application subnets. Its routes should remain private and use the NAT Gateway for outbound IPv4 access where required.

## 05. Internet Gateway attached

File: `screenshots/05-internet-gateway-attached.jpg`

The Internet Gateway is attached to the project VPC. An attachment alone is not enough; the public route table must also have a default route to this gateway.

## 06. Public default route added

File: `screenshots/06-public-route-to-internet-gateway.jpg`

The public route table shows 0.0.0.0/0 pointing to the Internet Gateway and the local VPC route. This is the route required for public-subnet internet traffic.

## 07. NAT Gateway created

File: `screenshots/07-nat-gateway-created.jpg`

The NAT Gateway was created for private-subnet outbound connectivity. Verify that its state is Available and that the private route table points its default route to the NAT Gateway before relying on it.

## 08. Load balancer security group

File: `screenshots/08-load-balancer-security-group.jpg`

LB-SG allows HTTP on port 80 and HTTPS on port 443 from the internet in the captured configuration. HTTPS requires a configured TLS listener and certificate; otherwise use the HTTP listener for this lab.

## 09. Application server security group

File: `screenshots/09-application-server-security-group.jpg`

APP-SERVER-SG allows inbound HTTP on port 80 from the load balancer security group, rather than from every internet address. This limits direct access to the application tier.

## 10. Database security group

File: `screenshots/10-database-security-group.jpg`

DB-SG is intended to allow MySQL on port 3306 only from APP-SERVER-SG. The screenshot shows the security group was created; ensure its inbound rule is added before database connectivity testing.

## 11. IAM role for EC2 Systems Manager

File: `screenshots/11-iam-ssm-role-created.jpg`

The IAM role list shows EC2SSMAgent was created. An EC2 instance needs the appropriate role/policy attached to use Systems Manager Session Manager.

## 12. Application EC2 instance

File: `screenshots/12-application-ec2-instance.jpg`

The EC2 console shows the project application instance running. Check its subnet, instance profile, security group, and status checks before configuring the web server.

## 13. Operating system and disk check

File: `screenshots/13-instance-os-and-storage-check.jpg`

The terminal output identifies Amazon Linux 2023 and displays filesystem usage. This confirms the shell session works and provides a basic disk-space check.

## 14. Apache service running

File: `screenshots/14-apache-service-running.jpg`

The terminal output shows the Apache httpd service is active and listening on port 80. This is the web-server layer that serves the WordPress files.

## 15. WordPress files on EC2

File: `screenshots/15-wordpress-files-on-ec2.jpg`

The /var/www/html listing shows WordPress application files on the EC2 instance. The application files alone do not prove database configuration or successful end-to-end access.

## 16. RDS DB subnet group

File: `screenshots/16-rds-db-subnet-group.jpg`

The private-db-subnet-group was created for the database tier. It groups the private database subnets so RDS can place the DB network interfaces in the VPC.

## 17. RDS MySQL instance available

File: `screenshots/17-rds-mysql-instance-available.jpg`

The RDS console shows wordpress-db in Available state. The database is managed by RDS and uses the private DB subnet group.

## 18. WordPress database verified

File: `screenshots/18-wordpress-database-verified.jpg`

The terminal shows a successful MySQL connection and the wordpress database in SHOW DATABASES output. The earlier CREATE DATABASE attempt reported that it already existed, so it was not necessary to create it again.

## 19. WordPress database user and grants

File: `screenshots/19-wordpress-db-user-and-privileges.jpg`

The terminal confirms the dedicated wp_user was created and granted privileges on wordpress.*. The original password is not reproduced in this repository; use a strong private password and never commit it.

## 20. Application Load Balancer active

File: `screenshots/20-application-load-balancer-active.jpg`

The 3tire-lb Application Load Balancer is active and internet-facing across two Availability Zones. Use its DNS name to test the application; the screenshot’s actual account/DNS details are intentionally redacted in the public image.

## 21. WordPress installation form

File: `screenshots/21-wordpress-installation-form.jpg`

The ALB address reached the WordPress installation screen. The screenshot has personal/login fields redacted. Do not publish the administrator password or email address in GitHub.

## 22. EC2 launch template created

File: `screenshots/22-launch-template-created.jpg`

The 3tire-app-launch-template was created for repeatable EC2 instance launches. Verify that the AMI, instance type, security group, IAM instance profile, and user data match the intended application setup.

## 23. Auto Scaling Group healthy

File: `screenshots/23-auto-scaling-group-healthy.jpg`

The 3tire-ASG is at its configured desired capacity and the captured console shows one healthy instance. This is a snapshot of the group at that time, not proof of a scaling test.

## 24. CloudWatch CPU alarm created

File: `screenshots/24-cloudwatch-cpu-alarm-created.jpg`

The 3tire-cpu-high CloudWatch alarm was created. Verify the alarm action, confirm the SNS endpoint, and test the notification before relying on alerts.

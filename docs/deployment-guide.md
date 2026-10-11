# Deployment Guide

This guide follows the project PDF's EC2-focused workflow. The resource names and CIDRs below match the lab screenshots where applicable. Avoid copying AWS-generated IDs, endpoints, passwords, or key material into public documentation.

## 1. Network foundation

1. Create or select a VPC with CIDR `10.0.0.0/16`.
2. Create two public subnets (`10.0.1.0/24`, `10.0.2.0/24`), two private application subnets (`10.0.3.0/24`, `10.0.4.0/24`), and two private DB subnets (`10.0.5.0/24`, `10.0.6.0/24`). Use separate Availability Zones for the pairs intended for the ALB and RDS subnet group.
3. Attach an Internet Gateway to the VPC.
4. Associate the public route table with both public subnets and add `0.0.0.0/0 → Internet Gateway`.
5. Place the NAT Gateway in a public subnet and associate an Elastic IP. Associate the private application route table with both app subnets and add `0.0.0.0/0 → NAT Gateway` once the NAT Gateway is available. Keep the DB route table private; the local VPC route is sufficient for the basic DB design.

## 2. Security groups

- `LB-SG`: allow inbound HTTP `80` from the internet for the temporary lab test. Configure HTTPS `443` only after configuring TLS.
- `APP-SERVER-SG`: allow inbound HTTP `80` from `LB-SG` only. Do not assign a public IPv4 address to the app instance.
- `DB-SG`: allow inbound MySQL `3306` from `APP-SERVER-SG` only.
- Administrative access should be restricted to a trusted path and source. Never allow SSH `22` from `0.0.0.0/0`.

## 3. Application EC2 and WordPress

Use Amazon Linux 2023. Install Apache, PHP, and supporting extensions:

```bash
sudo dnf update -y
sudo dnf install -y httpd php php-mysqli php-fpm php-json php-xml php-mbstring unzip wget
sudo systemctl enable --now httpd
```

Download and extract WordPress:

```bash
cd /var/www/html
sudo wget https://wordpress.org/latest.zip
sudo unzip latest.zip
sudo cp -a wordpress/. .
sudo rm -rf wordpress latest.zip
```

Set appropriate ownership and permissions, then verify Apache:

```bash
sudo chown -R apache:apache /var/www/html
sudo find /var/www/html -type d -exec chmod 755 {} \;
sudo find /var/www/html -type f -exec chmod 644 {} \;
sudo systemctl restart httpd
sudo systemctl status httpd --no-pager
```

Do not install a local database server on the application EC2 for this design; use RDS.

## 4. Amazon RDS for MySQL

1. Create a DB subnet group using `db-subnet1` and `db-subnet2` from different Availability Zones.
2. Create a MySQL RDS instance, select `DB-SG`, and set public access to **No**.
3. Wait until the database is available, then retrieve its endpoint from the RDS console.
4. From the app instance, use a MySQL-compatible client to connect to the endpoint. Create the WordPress database and a dedicated database user:

```sql
CREATE DATABASE wordpress
  DEFAULT CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

CREATE USER 'wp_user'@'%' IDENTIFIED BY 'REPLACE_WITH_A_STRONG_UNIQUE_PASSWORD';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wp_user'@'%';
FLUSH PRIVILEGES;
```

Use a strong unique password. Store real values securely and never commit them. Configure the application using the example in [`../config/wp-config.example.php`](../config/wp-config.example.php), setting the actual RDS endpoint as `DB_HOST` on the instance. The example file intentionally contains placeholders only.

## 5. Target group and Application Load Balancer

1. Create an instance target group using HTTP port `80` in the project VPC.
2. Configure the health-check path to `/wp-login.php` after WordPress files are in place.
3. Register the application instance and verify its target health.
4. Create an internet-facing Application Load Balancer using both public subnets and `LB-SG`.
5. Configure the HTTP `80` listener to forward to the target group.
6. Open the ALB's AWS-provided DNS name with `http://` to reach the WordPress installation page. This lab intentionally does not require Route 53, a custom domain, or ACM to demonstrate the traffic path.

## 6. Launch Template and Auto Scaling

1. Create a launch template from the tested application configuration or a reusable AMI; do not bake secrets into the image.
2. Create an Auto Scaling Group across the two private application subnets and attach it to the ALB target group.
3. Set capacity to fit the learning budget. The screenshots included with this project show one healthy instance at desired capacity.
4. Verify target health before retiring any original instance.

## 7. CloudWatch and SNS notifications

1. Create a CloudWatch alarm for the EC2 `CPUUtilization` metric and select the intended threshold and evaluation periods.
2. Connect the alarm action to an SNS topic.
3. Add the intended email/SMS endpoint and complete its subscription confirmation.
4. Test notification delivery before depending on it. Do not store phone numbers, recipient email addresses, or SNS credentials in GitHub.

## 8. Validation

Use [`validation-checklist.md`](validation-checklist.md) to verify the networking, application, database, ALB, scaling, and monitoring layers.

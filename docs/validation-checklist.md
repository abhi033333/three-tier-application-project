# Validation Checklist

Use this as a repeatable review before presenting the project.

## Networking

- [ ] VPC CIDR is `10.0.0.0/16`.
- [ ] All six subnets exist with the intended CIDRs and Availability Zone placement.
- [ ] Public route table sends `0.0.0.0/0` to the Internet Gateway.
- [ ] NAT Gateway is available and the private application route table sends `0.0.0.0/0` to it.
- [ ] Database route table remains private.

## Application

- [ ] Application EC2 has no public IPv4 address.
- [ ] Apache is running and enabled at boot.
- [ ] WordPress files are present in `/var/www/html`.
- [ ] `wp-config.php` has the correct RDS database name, dedicated username, password, and endpoint; secrets are not in the repository.

## Database

- [ ] RDS MySQL instance is available and is not publicly accessible.
- [ ] DB subnet group covers two DB subnets in different Availability Zones.
- [ ] `wordpress` database and the dedicated application database user exist.
- [ ] `DB-SG` allows MySQL `3306` from the application security group only.

## Load balancing and scaling

- [ ] ALB is active, uses both public subnets, and has an HTTP listener forwarding to the target group.
- [ ] Target group health check is healthy.
- [ ] Launch template uses the correct AMI/configuration and application security group.
- [ ] Auto Scaling Group spans the two private application subnets and is attached to the target group.

## Monitoring

- [ ] CloudWatch alarm monitors the intended EC2 CPU metric with reviewed threshold and evaluation periods.
- [ ] Alarm action points to the correct SNS topic.
- [ ] SNS email/SMS endpoint has confirmed the subscription and an end-to-end test is successful.

## Public GitHub review

- [ ] No passwords, private keys, endpoint recipient details, or real `wp-config.php` are committed.
- [ ] Screenshots do not expose account identifiers or personal contact details.
- [ ] Billing resources are reviewed after testing.

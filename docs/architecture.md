# Architecture Notes

The project separates responsibilities into three tiers:

1. **Public entry tier:** Internet Gateway and Application Load Balancer deployed across two public subnets.
2. **Application tier:** WordPress with Apache/PHP hosted on Amazon Linux EC2 instances in private subnets. A Launch Template and Auto Scaling Group manage instance configuration and desired capacity.
3. **Database tier:** Amazon RDS for MySQL in private database subnets, accessed from the application tier on port `3306`.

The private application subnets use a NAT Gateway for outbound package downloads and updates; this is separate from inbound application traffic. The ALB forwards inbound HTTP requests to the app target group. CloudWatch evaluates EC2 CPU utilization and can invoke an SNS topic for email/SMS notifications.

The lab guide uses the ALB's AWS-provided DNS name over HTTP and omits Route 53, custom-domain registration, and ACM certificate configuration. That is appropriate only for a temporary learning lab, not a production security design.

# Security and Cost Notes

## Security controls

- Keep application EC2 instances and RDS in private subnets.
- Permit application HTTP traffic only from the ALB security group.
- Permit database port `3306` only from the application security group.
- Restrict administrative access. Never expose SSH port `22` to all IPv4 addresses.
- Use IAM roles and least-privilege policies rather than embedded access keys.
- Do not commit `wp-config.php`, passwords, private keys, account credentials, RDS endpoints when not needed, phone numbers, or recipient email addresses.
- Rotate any real password that may have been exposed in screenshots or chat, even if the field was masked.
- The example uses HTTP for a temporary educational test. For production, configure HTTPS/TLS, a custom domain if required, patched software, backups, logging, and a reviewed security posture.

## Cost controls

This lab can incur charges. NAT Gateway, Application Load Balancer, RDS, public IPv4 addresses, EC2 compute, storage, and data transfer may be billable depending on region, account age, usage, and AWS pricing terms.

- Check Billing, Free Tier eligibility, and Cost Explorer before and during the lab.
- Stop or delete resources when they are not required.
- If deleting the lab, remove dependent resources in the correct order (for example, detach/delete the Auto Scaling group and load balancer before removing resources they depend on).
- Review the NAT Gateway and RDS lifecycle carefully; leaving them running may continue to generate charges.

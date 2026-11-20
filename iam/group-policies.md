# IAM group and policies

This project was built with a dedicated IAM user, not the root account.

| Item | Value |
|---|---|
| IAM user | `aws-ha-webapp-admin` |
| IAM group | `grp-aws-ha-webapp-project` |
| How permissions are granted | Policies are attached to the group. The user inherits them through group membership. |

Using a group means permissions can be changed in one place, and nothing is attached to the user directly.

---

## Policies attached to the group

All seven are AWS managed policies.

| # | Policy | Why it was needed |
|---|---|---|
| 1 | `AmazonEC2FullAccess` | Launch templates, EC2 instances (app servers and bastion), key pairs, security groups, Elastic IPs, target groups and load balancers (the console manages these under EC2). |
| 2 | `AmazonVPCFullAccess` | Create the VPC, subnets, route tables, Internet Gateway and NAT Gateways. |
| 3 | `AutoScalingFullAccess` | Create and manage the Auto Scaling Group. |
| 4 | `ElasticLoadBalancingFullAccess` | Create the Application Load Balancer, listeners and target group. |
| 5 | `CloudWatchReadOnlyAccess` | View metrics and alarms for the instances, ASG and load balancer. Read-only, no changes. |
| 6 | `IAMReadOnlyAccess` | View IAM users, groups, roles and policies from the console. Read-only, no changes. |
| 7 | `AWSHealthFullAccess` | View AWS Health events and notifications for the account. |

### What each group of policies covers

| Area | Policies |
|---|---|
| Build the network | `AmazonVPCFullAccess` |
| Build compute and scaling | `AmazonEC2FullAccess`, `AutoScalingFullAccess` |
| Build the load balancer | `ElasticLoadBalancingFullAccess` (plus `AmazonEC2FullAccess` for target groups) |
| Visibility only (no write access) | `CloudWatchReadOnlyAccess`, `IAMReadOnlyAccess`, `AWSHealthFullAccess` (health events are informational) |

---

## What this user cannot do

- It cannot create or modify IAM users, roles or policies (only read them).
- It cannot change billing or account settings.
- It cannot manage its own MFA device unless a policy allows it (see below).

---

## MFA note

None of the policies above allow a user to manage its own MFA device, so assigning MFA to this user failed with `iam:EnableMFADevice` not authorized.

**Fix:** assign the MFA device from the root account (done once, then sign out of root).

---

## Least privilege: what I would change in production

The `FullAccess` managed policies are convenient for a lab but broader than a real team should have.

| Area | Lab setup | Production approach |
|---|---|---|
| EC2, VPC, Auto Scaling, Load Balancing | `FullAccess` managed policies | Custom policies limited to the actions and resources the project needs, ideally with tags or resource ARNs as conditions |
| Account access | One IAM user with console access | IAM Identity Center (SSO) with short-lived credentials |
| MFA | Enabled per user | Enforced through a policy that denies actions unless MFA is present |
| Access keys | None created | Avoid long-lived keys, use roles |

---

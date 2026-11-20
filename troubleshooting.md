# Troubleshooting

Problems I ran into while building this project, with the cause and the fix for each. Account IDs in error messages are replaced with `123456789012`.

| # | Problem | Area |
|---|---|---|
| 1 | IAM user could not enable MFA | IAM |
| 2 | VPC creation failed: Elastic IP limit reached | VPC / NAT Gateway |

A short diagnosis table for SSH and load balancer errors is at the end.

---

## 1. IAM user could not enable MFA

**Symptom**

Assigning an MFA device to the IAM user `aws-ha-webapp-admin` failed with:

```
User: arn:aws:iam::123456789012:user/aws-ha-webapp-admin is not authorized
to perform: iam:EnableMFADevice on resource: user aws-ha-webapp-admin
because no identity-based policy allows the iam:EnableMFADevice action
```

**Cause**

The group policies (EC2, VPC, Auto Scaling, Elastic Load Balancing, and the read-only policies) gave the user permissions for the project services, but none of them allowed the user to manage its own MFA device. AWS denies anything that isn't explicitly allowed.

**Fix**

- Assign the MFA device from the root account, then sign back in as the IAM user.

**What I learned**

A user can be allowed to build infrastructure and still be unable to manage its own credentials. Reading the "not authorized" message, which names the exact action that was denied, tells you which permission to add.

---

## 2. VPC creation failed: Elastic IP limit reached

**Symptom**

Creating the VPC with the "VPC and more" wizard failed while creating the NAT Gateways, with the message:

```
The maximum number of addresses has been reached
```

**Cause**

Each NAT Gateway needs its own Elastic IP, and this project uses two (one per AZ). The account already had Elastic IPs from earlier labs, and the default limit is 5 per Region. The new NAT Gateways could not get an address.

**Fix**

1. EC2 → Network & Security → **Elastic IPs**.
2. Select the addresses that had no association.
3. **Actions → Release Elastic IP addresses**.
4. Re-run the VPC creation.

**What I learned**

Unattached Elastic IPs from old labs also count toward the limit, and they can be billed. Release anything you no longer use after a lab.

---


## Quick diagnosis reference

Common symptoms in this setup and where to look first.

| Symptom | Likely cause | Check |
|---|---|---|
| `ssh` to the bastion or a private instance **hangs** | Security group blocks port 22 | Bastion SG allows 22 from my IP. Instance SG allows 22 from the bastion SG. |
| `Permission denied (publickey)` | Wrong key, wrong user, or key permissions too open | User is `ubuntu`. Run `chmod 400` on the `.pem` file. Use the same key pair the instance was launched with. |
| Can't reach a private instance by public DNS | Private instances have no public IP | Connect by private IP from the bastion. |
| Browser shows **503** from the load balancer | No healthy targets | Target group shows both targets healthy. App is running on port 8000. |
| Browser **times out** on the ALB URL | ALB security group blocks HTTP | `alb-sg` allows inbound port 80 from `0.0.0.0/0`. |
| Target stays **unhealthy** even though the app runs | Instance SG blocks the ALB | Instance SG allows port 8000 from `alb-sg`. Health check path is `/`. |
| Private instance can't download packages | No route to a NAT Gateway | Private route table has `0.0.0.0/0` pointing to the NAT Gateway in the same AZ. |

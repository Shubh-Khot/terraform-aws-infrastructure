# Terraform AWS Infrastructure

Production-ready AWS infrastructure using Terraform modules. Provisions a complete 3-tier architecture with VPC, EC2, RDS (PostgreSQL), and S3.

## Architecture

```
                        Internet
                           │
                    [Internet Gateway]
                           │
              ┌────────────┴────────────┐
              │        Public Subnets    │
              │   [NAT GW]  [EC2 Bastion]│
              └────────────┬────────────┘
                           │
              ┌────────────┴────────────┐
              │       Private Subnets   │
              │    [RDS PostgreSQL]     │
              └─────────────────────────┘
```

## Modules

| Module | Description |
|--------|-------------|
| `vpc`  | VPC, subnets, IGW, NAT Gateways, route tables |
| `ec2`  | Bastion host with security group |
| `rds`  | PostgreSQL RDS with Multi-AZ support for prod |
| `s3`   | Encrypted, versioned S3 bucket |

## Prerequisites

- [Terraform](https://www.terraform.io/downloads) >= 1.5.0
- [AWS CLI](https://aws.amazon.com/cli/) configured with appropriate credentials
- S3 bucket for remote state (update `backend` in `main.tf`)
- DynamoDB table for state locking

## Usage

### Deploy to Dev

```bash
cd environments/dev
terraform init
terraform plan -var-file=terraform.tfvars -var="key_name=my-key" -var="db_username=admin" -var="db_password=secret"
terraform apply -var-file=terraform.tfvars -var="key_name=my-key" -var="db_username=admin" -var="db_password=secret"
```

### Deploy to Prod

```bash
cd environments/prod
terraform init
terraform plan -var-file=terraform.tfvars -var="key_name=my-key" -var="db_username=admin" -var="db_password=secret"
terraform apply -var-file=terraform.tfvars -var="key_name=my-key" -var="db_username=admin" -var="db_password=secret"
```

### Destroy

```bash
terraform destroy -var-file=terraform.tfvars -var="key_name=my-key" -var="db_username=admin" -var="db_password=secret"
```

## Security Considerations

- RDS is deployed in private subnets only
- S3 bucket has public access blocked and encryption enabled
- Remote state is stored encrypted in S3 with DynamoDB locking
- Sensitive variables (`db_username`, `db_password`) are marked sensitive and should be passed via environment variables or a secrets manager

## Environments

| Feature           | Dev         | Prod          |
|-------------------|-------------|---------------|
| RDS Multi-AZ      | No          | Yes           |
| Deletion Protection | No        | Yes           |
| Backup Retention  | 1 day       | 7 days        |
| Instance Type     | t3.micro    | t3.small      |

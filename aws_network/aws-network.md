# Plan: VPC Infrastructure in new_account_vpc/vpc.yaml

## Context
Write fresh Crossplane YAML using `crossplane-contrib/provider-aws v0.58.0` to provision foundational AWS network infrastructure in `us-west-2`. The file is currently empty.

## Resources to Create (all in one multi-document YAML file)

All resources use `ec2.aws.crossplane.io/v1beta1` (confirmed from CRD inspection).

| # | Kind | Name | Notes |
|---|------|------|-------|
| 1 | VPC | `new-account-vpc` | 10.80.0.0/16, DNS enabled |
| 2 | InternetGateway | `new-account-igw` | attached to VPC |
| 3 | Subnet (public) | `new-account-public-subnet-1..4` | 10.80.0-3.0/24, AZs a/b/c/d, mapPublicIpOnLaunch: true |
| 4 | Subnet (private) | `new-account-private-subnet-1..4` | 10.80.10-13.0/24, AZs a/b/c/d |
| 5 | Address (EIP) | `new-account-nat-eip` | domain: vpc |
| 6 | NATGateway | `new-account-nat-gw` | in public-subnet-1, refs EIP via allocationIdRef |
| 7 | RouteTable (public) | `new-account-public-rt` | route 0.0.0.0/0 → IGW, assoc all 4 public subnets |
| 8 | RouteTable (private) | `new-account-private-rt` | route 0.0.0.0/0 → NAT GW, assoc all 4 private subnets |

**Total: 14 resources** (1 VPC + 1 IGW + 8 Subnets + 1 EIP + 1 NAT GW + 2 Route Tables)

## CIDR Layout
- VPC: `10.80.0.0/16`
- Public subnets: `10.80.0.0/24` (2a), `10.80.1.0/24` (2b), `10.80.2.0/24` (2c), `10.80.3.0/24` (2d)
- Private subnets: `10.80.10.0/24` (2a), `10.80.11.0/24` (2b), `10.80.12.0/24` (2c), `10.80.13.0/24` (2d)

## NAT Gateway Design
Single NAT Gateway in `public-subnet-1` (us-west-2a). All private subnets share one private route table pointing to it. Cost-efficient single-AZ NAT.

## Critical File
- **Target:** `new_account_vpc/vpc.yaml` (overwrite with full multi-doc YAML)

## Verification
After applying: `kubectl get vpc,subnet,internetgateway,natgateway,routetable,address -l app=new-account-vpc`
All resources should reach `SYNCED=True` and `READY=True`.

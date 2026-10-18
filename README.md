# Terraform Cloud Integration Lab

This project demonstrates how to connect a local Terraform configuration to Terraform Cloud, use remote state management, execute remote plans and applies, manage workspace variables, and collaborate through Terraform Cloud runs.

## Overview

The lab covers:

- Installing Terraform and Git on a Linux machine
- Creating a Terraform Cloud account, organization, and API token
- Authenticating the Terraform CLI with Terraform Cloud
- Configuring a Terraform Cloud workspace with remote execution
- Pushing configurations and running remote plans/applies
- Using workspace variables and reviewing run history
- Cleaning up resources and verifying remote state

## Prerequisites

- Basic Terraform concepts: resources, providers, state
- Linux command line familiarity
- Basic Git knowledge
- Infrastructure as Code (IaC) understanding
- GitHub account (free tier)
- Terraform Cloud account (free tier)

## Project Structure

```text
terraform-cloud-lab/
├── configurations/
│   ├── main.tf
│   ├── variables.tf
│   ├── advanced.tf
│   ├── terraform.tfvars.example
│   └── .gitignore
├── scripts/
├── docs/
└── README.md
```

The Terraform CLI credentials file is stored outside the project:

```text
~/.terraform.d/credentials.tfrc.json
```

## 1. Environment Setup

Install required tools:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget unzip git
wget https://releases.hashicorp.com/terraform/1.6.6/terraform_1.6.6_linux_amd64.zip
unzip terraform_1.6.6_linux_amd64.zip
sudo mv terraform /usr/local/bin/
rm terraform_1.6.6_linux_amd64.zip
terraform version
```

Configure Git:

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --list
```

Create the project directory and initialize Git:

```bash
mkdir -p ~/terraform-cloud-lab
cd ~/terraform-cloud-lab
mkdir -p configurations scripts docs
git init
```

## 2. Terraform Cloud Setup

1. Sign in to https://app.terraform.io.
2. Create or select an organization.
3. Open User Settings > Tokens.
4. Create an API token named `Lab 18 Token`.
5. Copy the token securely.

Configure CLI authentication:

```bash
mkdir -p ~/.terraform.d
nano ~/.terraform.d/credentials.tfrc.json
```

Use this structure:

```json
{
  "credentials": {
    "app.terraform.io": {
      "token": "YOUR_TERRAFORM_CLOUD_TOKEN_HERE"
    }
  }
}
```

Replace `YOUR_TERRAFORM_CLOUD_TOKEN_HERE` with the real token.

## 3. Terraform Configuration

The main configuration uses the Terraform Cloud backend:

```hcl
terraform {
  required_version = ">= 1.0"

  cloud {
    organization = "YOUR_ORG_NAME"

    workspaces {
      name = "terraform-cloud-lab"
    }
  }
}
```

Before running Terraform, replace `YOUR_ORG_NAME` in `configurations/main.tf` with your Terraform Cloud organization name.

The configuration creates:

- A random string lab ID
- A local text output file
- A random password and UUID
- A JSON output file
- Terraform outputs for the lab ID, UUID, and generated files

## 4. Initialize and Create the Workspace

```bash
cd ~/terraform-cloud-lab/configurations
terraform init
```

When prompted, confirm creation of the Terraform Cloud workspace.

In Terraform Cloud, open the workspace and set:

- Execution Mode: Remote
- Apply Method: Manual apply

Add workspace variables:

- `environment` = `development`
- `project_name` = `terraform-cloud-lab`

## 5. Remote Plan and Apply

Run a remote plan:

```bash
terraform plan
```

Review the run in Terraform Cloud under Runs.

Apply the configuration:

```bash
terraform apply
```

Type `yes` when prompted.

Verify state and outputs:

```bash
terraform show
terraform state list
terraform state show random_string.lab_id
terraform output
terraform output lab_id
terraform output lab_uuid
```

Inspect generated files:

```bash
ls -la
cat lab-output-*.txt
cat advanced-*.json | python3 -m json.tool
```

## 6. Advanced Configuration

`configurations/advanced.tf` adds:

- `random_password`
- `random_uuid`
- A JSON local file containing lab metadata and security details
- Additional outputs

Apply the advanced configuration with:

```bash
terraform plan
terraform apply
```

## 7. Workspace Variables and Run History

Create an example variables file:

```bash
cp terraform.tfvars.example terraform.tfvars
```

In Terraform Cloud, update the `environment` workspace variable to `production`, then run:

```bash
terraform plan
```

Workspace variables override local defaults.

Review run history in Terraform Cloud under Runs to compare changes and audit operations.

## 8. Git Workflow

Add and commit the configuration:

```bash
git add .
git commit -m "Initial Terraform Cloud configuration"
git add .gitignore
git commit -m "Add .gitignore file"
```

Final commit after cleanup:

```bash
git add .
git commit -m "Complete Terraform Cloud integration lab"
git log --oneline
```

## 9. Cleanup

Plan and execute destroy:

```bash
terraform plan -destroy
terraform destroy
```

Verify cleanup:

```bash
ls -la *.txt *.json
terraform show
```

## Troubleshooting

### Authentication Problems

```bash
cat ~/.terraform.d/credentials.tfrc.json
terraform login
```

### Workspace Not Found

- Verify the organization name in `configurations/main.tf`.
- Check the workspace name spelling.
- Confirm the workspace exists in Terraform Cloud.

### Remote Execution Failures

- Confirm Execution Mode is set to `Remote`.
- Confirm required workspace variables exist.
- Review run logs in the Terraform Cloud UI.

### State Lock Issues

```bash
terraform force-unlock LOCK_ID
```

Use force-unlock with caution.

## Key Concepts

- Remote state management with locking and versioning
- Remote execution for consistent environments
- Workspace variables for configuration and secrets
- Run history and auditability
- Collaboration through Terraform Cloud workflows
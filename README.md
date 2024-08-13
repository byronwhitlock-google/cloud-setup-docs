<span _ngcontent-ng-c3468594053="" data-testid="p6ntest-llm-prompt-response-text-area-text-segment" sandboxuid="0" class="ng-star-inserted">```markdown
# Google Cloud Foundation Blueprint

This repository contains Terraform configuration files based on Google Cloud best practices to deploy a foundational environment on Google Cloud.  It provisions resources adhering to the recommendations outlined in the [setup checklist](https://cloud.google.com/docs/enterprise/setup-checklist).   

## Architecture

This blueprint sets up the following:

- **Organization:** Establishes the root node of your Google Cloud resource hierarchy, enabling centralized management. [Learn more](https://cloud.google.com/resource-manager/docs/creating-managing-organization).
- **Folders:** Creates a hierarchical structure under your organization for logical grouping of projects (e.g., "Production," "Non-Production," "Development"). [Learn more](https://cloud.google.com/resource-manager/docs/creating-managing-folders).
- **Projects:** Provisions projects within the defined folders for isolating resources and applying specific policies. [Learn more](https://cloud.google.com/resource-manager/docs/creating-managing-projects).
- **Shared VPC:** Sets up a Shared VPC for centralized network management and efficient resource utilization across projects. [Learn more](https://cloud.google.com/vpc/docs/shared-vpc).
- **Networking:** Configures VPC networks and subnets, including firewall rules for secure communication. [Learn more](https://cloud.google.com/vpc/docs/using-vpc).
- **IAM:** Defines Identity and Access Management roles and bindings at the organization, folder, and project levels to control access to resources. [Learn more](https://cloud.google.com/iam/docs/overview).
- **Groups:** Leverages Google Groups to simplify IAM management by assigning permissions to groups instead of individual users. [Learn more](https://support.google.com/a/answer/2405986).
- **Centralized Logging and Monitoring:** Deploys a centralized project for collecting and analyzing logs and metrics from other projects, enhancing visibility and troubleshooting capabilities. [Learn more](https://cloud.google.com/logging/docs/overview) and [Learn more](https://cloud.google.com/monitoring/docs/overview).

## Prerequisites

Before deploying this blueprint, ensure you have the following:

- **Google Cloud Account:** A Google Cloud Platform account with billing enabled. [Sign up for a free trial](https://cloud.google.com/free).
- **Terraform:** Install Terraform (version 0.13.7 or later) to manage your infrastructure as code. [Download Terraform](https://www.terraform.io/downloads.html).
- **Permissions:** Ensure the user executing the Terraform deployment has the following roles:
    - `roles/billing.admin` on the billing account.
    - `roles/resourcemanager.organizationAdmin` on the Google Cloud organization.
    - `roles/resourcemanager.folderCreator` on the Google Cloud organization.
    - `roles/resourcemanager.projectCreator` on the Google Cloud organization.
    - `roles/compute.xpnAdmin` on the Google Cloud organization.
    - Group Admin role in Google Admin (for creating Google Groups).
    - TODO: Add extra permssisons needed here
    - TODO: Is there a bootstrap terraform/script

## Committing to GitHub

1. **Initialize a Git repository (if not already done):**

```bash
git init
```

2. **Add your Terraform files:**

```bash
git add .
```

3. **Commit your changes:**

```bash
git commit -m "Initial commit of Terraform foundation blueprint"
```

4. **Create a GitHub repository and push your local repository:**

```bash
git remote add origin <your-github-repository-url>
git push -u origin main
```

## Manual Deployment

1. **Clone the Repository:** Clone this repository to your local machine.

   ```bash
   git clone <your-github-repository-url>
   cd <your-github-repository>
   ```

2. **Set up Authentication:** Configure your Google Cloud credentials for Terraform to access your Google Cloud resources. [Learn more](https://registry.terraform.io/providers/hashicorp/google/latest/docs#authentication).

3. **Update Variables:** Verify the `variables.tf` file contains your desired organization ID and billing account number.

   ```terraform
   variable "billing_account" {
     description = "The ID of the billing account to associate projects with"
     type        = string
     default     = "<please enter your billing account number here>"
   }

   variable "org_id" {
     description = "The organization id for the associated resources"
     type        = string
     default     = "<Your Organization ID>" 
   }
   ```

4. **Initialize Terraform:** Initialize your Terraform working directory.

   ```bash
   terraform init
   ```

5. **Review the Plan:** Generate an execution plan and carefully review the changes Terraform will make.

   ```bash
   terraform plan
   ```

6. **Apply the Configuration:** Apply the Terraform configuration to deploy your infrastructure.

   ```bash
   terraform apply
   ```


## Setting up CI/CD with Cloud Build

Follow these steps to set up automated deployments with Cloud Build:
TODO: FIX THESE STEPS WITH PSO RECOMMENDED EXAMPLES FROM DELNAV

1. **Enable the Cloud Build API:** Enable the [Cloud Build API](https://console.cloud.google.com/apis/library/cloudbuild.googleapis.com) in your Google Cloud project.

2. **Create a Cloud Build Trigger:** Create a new trigger in the Cloud Build console and configure it as follows:
   - **Name:** Give your trigger a descriptive name (e.g., `terraform-foundation-deploy`).
   - **Event:** Select "Push to a branch."
   - **Source:** Select your GitHub repository and the branch you want to trigger deployments from.
   - **Configuration:** Choose "Cloud Build configuration file" and specify `cloudbuild.yaml` as the path.

3. **Create a `cloudbuild.yaml` File:** Create a file named `cloudbuild.yaml` in the root of your repository with the following contents:

   ```yaml
   steps:
   - name: 'hashicorp/terraform:latest'
   args: ['init']
   - name: 'hashicorp/terraform:latest'
   args: ['plan']
   - name: 'hashicorp/terraform:latest'
   args: ['apply', '-auto-approve']
   ```

   **Explanation:**
      - This configuration defines three build steps using the official Terraform Cloud Builder image.
      - It runs `terraform init` to initialize the workspace.
      - It runs `terraform plan` to generate an execution plan.
      - It runs `terraform apply -auto-approve` to automatically apply the changes.


4. **Grant Cloud Build Service Account Permissions:** The Cloud Build service account needs appropriate permissions to deploy resources. Grant it the necessary roles based on the resources defined in your Terraform configuration. You can find more information in the Cloud Build [documentation on service accounts](https://cloud.google.com/build/docs/securing-builds/configure-user-accounts).

5. **Test Your Pipeline:** Push a commit to your GitHub repository to trigger the Cloud Build pipeline. You can monitor the build progress and logs in the Cloud Build console.

## Service Account Best Practices

This section outlines best practices for the service account used to run this Terraform code, emphasizing the principle of least privilege ([Learn More](https://cloud.google.com/iam/docs/best-practices-access-management#principle-of-least-privilege)).

**Instead of granting overly broad permissions at the organization level, follow these recommendations:**

1. **Dedicated Service Account:** Create a dedicated service account specifically for managing your infrastructure with Terraform.
TODO: CHECK DOES THIS ALREADY HAPPEN IN CURRENT CLOUD SETUP??????

2. **Temporary Credentials:** Use temporary credentials, such as those obtained from Cloud SDK or Workload Identity Federation, to avoid storing long-lived service account keys. [Learn More](https://cloud.google.com/iam/docs/best-practices-for-managing-service-account-keys)

4. **Role Separation:** Consider using separate service accounts for different tasks e.g., one for this deployment, another for application deployments. This further limits the blast radius of potential compromises. 



## Next Steps

- **Customize:** Tailor the provided configurations (e.g., network settings, IAM bindings, project quotas) to meet your specific requirements.
- **Advanced Foundation:** Explore building upon this basic foundation by incorporating additional security controls and best practices from the [advanced foundation blueprint](https://github.com/terraform-google-modules/terraform-example-foundation).
- **Security Foundation:** Learn more about security foundations in the [security foundations blueprint](https://cloud.google.com/architecture/security-foundations).
- **Monitoring and Logging:** Integrate monitoring and logging solutions to gain visibility into your deployed resources and troubleshoot potential issues.
- ***Deeper Automation with Cloud Build:** Explore further automation capabilities with Cloud Build, such as:
   - Running automated tests.
   - Implementing approval gates for production deployments.
   - Integrating with other DevOps tools.


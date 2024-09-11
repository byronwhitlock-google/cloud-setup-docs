# Google Cloud Foundation Blueprint


This repository contains Terraform configuration files based on Google Cloud best practices to deploy a foundational environment on Google Cloud.  It provisions resources adhering to the recommendations outlined in the [setup checklist](https://cloud.google.com/docs/enterprise/setup-checklist).  For a complete set of instructions, please refer to the [official deployment guide](https://cloud.google.com/docs/enterprise/deploy-foundation-using-terraform-from-console)


## Architecture


This blueprint  was automatically generated by GCP Cloud Setup.The specific resources in your Terraform module can vary based on your initial cloud configuration decisions.


- **`folders`:**
  - Creates a hierarchical structure under your organization for logical grouping of projects (e.g., "Production," "Non-Production," "Development"). [Learn more](https://cloud.google.com/resource-manager/docs/creating-managing-folders).
- **`projects`:**
  - Provisions projects within the defined folders for isolating resources and applying specific policies. [Learn more](https://cloud.google.com/resource-manager/docs/creating-managing-projects).
- **`network`:**
  - Configures VPC networks and subnets, including firewall rules for secure communication. [Learn more](https://cloud.google.com/vpc/docs/using-vpc).
  - Configures Shared VPC for centralized network management and efficient resource utilization across projects. [Learn more](https://cloud.google.com/vpc/docs/shared-vpc).
- **`iam`:**
  - Defines Identity and Access Management roles and bindings at the organization, folder, and project levels to control access to resources. [Learn more](https://cloud.google.com/iam/docs/overview).
- **`groups`:**
  - Leverages Google Groups to simplify IAM management by assigning permissions to groups instead of individual users. [Learn more](https://support.google.com/a/answer/2405986).
- **`service-projects`:**
  -  Creates individual service accounts for Prod and Non Prod environments. [Learn more](https://cloud.google.com/docs/enterprise/best-practices/establish-projects).
- **`vpn`:**
  - Creates hybrid connectivity to your on premises networks using a optional highly available VPN. [Learn more](https://cloud.google.com/network-connectivity/docs/vpn/concepts/best-practices)
- **`log-export`:**
  - Deploys a centralized project for collecting and analyzing logs and metrics from other projects, enhancing visibility and troubleshooting capabilities. [Learn more about logging](https://cloud.google.com/logging/docs/overview) and [Learn more about monitoring](https://cloud.google.com/monitoring/docs/overview).
- **`org-policy`:** 
  - Configures security using Org Policies for centralized control and Security Command Center for threat detection and response. [Org Policies](https://cloud.google.com/resource-manager/docs/organization-policy/overview) [Security Command Center](https://cloud.google.com/security-command-center/docs/optimize-security-command-center)
- **`monitoring`:** 
   - Centralizes metrics scope so that metrics from all of your projects can be viewed through a single scoping project. [Learn more](https://cloud.google.com/monitoring/settings).
   
- **`groups`:** 
   - Groups who will be able to use the service projects. [Learn more](https://cloud.google.com/identity/docs/groups).


## Prerequisites
Before deploying this blueprint, ensure you have the following:
- **Google Cloud Account:** A Google Cloud Platform account with billing enabled.
- **GCP Cloud Shell:**  Click  "Activate Cloud Shell" at the top of the Google Cloud console or navigate to [shell.cloud.google.com](https://shell.cloud.google.com).
- **Active Billing Account** If you haven't set up a working billing account at cloud-setup/billing, please refer to the Bootstrap section in the appendix.

- **State File:** This blueprint utilizes a Google Cloud Storage (GCS) backend to store the Terraform state file. Cloud Setup usually creates a GCS bucket for you as part of generating this blueprint. If the bucket in your `backends.tf` file does not point to a valid GCS bucket, you will need to create one yourself. See the Bootstrap step in the Appendix.

- **Git repository:** Code should be committed to a git repository such as GCP Secure Source Manager,  GitHub, AzureDevOps or similar depending on your needs. For detailed instructions, see Git Repository in the appendix

- **Update Variables:** Verify the `cloud-setup.auto.tfvars` file contains your desired organization ID and billing account number.
  ```terraform
      org_id          = "<please confirm your organization id here>"
      billing_account = "<please confirm your billing account number here>"
  ```

## Deployment
To run this Terraform code, you'll need to use either a service account or a user account. This account must have the appropriate IAM roles to provision the resources defined in the blueprint and needs to be authenticated with Google Cloud.
### Option 1: Impersonate Service Account
   This option allows Cloud Shell to impersonate a service account with the necessary permissions to deploy your Terraform infrastructure. This approach aligns with the principle of least privilege, as recommended by [Google Cloud's security best practices](https://cloud.google.com/security/best-practices).
   **Steps:**
   1. **Create a Service Account:**
      If the terraform deployer service account has not been created:
      ```bash
      gcloud iam service-accounts create terraform-deployer --display-name "Terraform Deployer"
      ```

   2. **Grant Necessary IAM Permissions:**
      - Grant permission for your user to impersonate the newly created service account. Your will need the `roles/iam.serviceAccountUser` permission to run this command. [Learn More](https://cloud.google.com/iam/docs/service-account-permissions)
      ```
      gcloud iam service-accounts add-iam-policy-binding "serviceAccount:terraform-deployer@PROJECT_ID.iam.gserviceaccount.com" \
      --member="user:<YOUR_GCP_EMAIL>" \
      --role="roles/iam.serviceAccountTokenCreator"
      ```

      - Assign Required Roles below:
      - `roles/billing.projectManager`
      - `roles/billing.user`
      - `roles/compute.xpnAdmin`
      - `roles/config.agent`
      - `roles/logging.configWriter`
      - `roles/orgpolicy.policyAdmin`
      - `roles/resourcemanager.folderIamAdmin`
      - `roles/resourcemanager.folderCreator`
      - `roles/resourcemanager.folderEditor`
      - `roles/resourcemanager.projectIamAdmin`
      - `roles/resourcemanager.projectCreator`
      - `roles/resourcemanager.projectDeleter`
      - `roles/serviceusage.serviceUsageConsumer`
      - `roles/secretmanager.secretAccessor`
      - `roles/iam.serviceAccountUser`
      - `roles/storage.objectUser`
      
      ```
      gcloud organizations add-iam-policy-binding ORGANIZATION_ID \
      --member="serviceAccount:terraform-deployer@PROJECT_ID.iam.gserviceaccount.com" \
      --role="ROLE_NAME"   
      ```
   
   3. **Setup Service Account Impersonation:**
      ```bash
      gcloud auth application-default login --impersonate-service-account terraform-deployer@PROJECT_ID.iam.gserviceaccount.com
      ```

   4. **Initialize and Apply Terraform:**
      - Run `terraform init` and `terraform apply` as usual. The gcloud CLI will automatically use the impersonated service account's credentials for authentication with GCP.

   - **Note:** After you're done with the deployment, it's recommended to revoke the impersonation by closing the terminal session or running `gcloud auth revoke`.

### Option 2: Bastion Host
   This approach involves creating a dedicated Compute Engine VM (bastion host) within your VPC network to manage your Terraform deployments. This method is useful when you need to deploy resources within a private network or have strict security policies that restrict direct access from external environments. [Learn More.](https://cloud.google.com/vpc/docs/using-bastion-hosts)

   **Steps:**
   1. **Create a Bastion VM:**
      ```bash
      gcloud compute instances create bastion-host \
      --zone=ZONE \
      --machine-type=e2-medium \
      --image-family=debian-cloud \
      --image-project=debian-cloud \
      --boot-disk-size=50GB \
      --subnet=SUBNET_NAME \
      --tags=bastion
      ```
      Replace `ZONE`, `SUBNET_NAME` with your desired values. Ensure the subnet has appropriate firewall rules to allow SSH access.

   2. **Grant Necessary IAM Permissions to the Bastion VM's Default Service Account:**
      - Similar to Option 1, determine the required IAM roles for your deployment and grant them to the default service account associated with the bastion host. You can find the service account email in the VM instance details in the GCP Console.

   3. **Connect to the Bastion VM:**
      ```bash
      gcloud compute ssh bastion-host --zone=ZONE
      ```

   4. **Initialize and Apply Terraform:**
      - Once connected to the bastion host, run `terraform init` and `terraform apply`. Terraform will automatically use the VM's default service account for authentication.

### Option 3: User Cloud Shell Directly or Local Machine (Pre-Authenticated)

This option assumes that your user account (in Cloud Shell or locally) already has the necessary IAM permissions for the Terraform deployment. This approach is convenient for quick deployments or experimentation but can pose security risks if your user account has excessive privileges.

**Steps (using Cloud Shell):**

   1. **Open Cloud Shell:** Go to the GCP console and click the Cloud Shell icon in the top right corner.

   2. **Verify Permissions:**
   - Check your user account's IAM permissions using:
   ```bash
   gcloud organizations get-iam-policy ORG_ID
   ```
   - Ensure you have the required roles for your Terraform deployment.

   3. **Initialize and Apply Terraform:**
   - Navigate to your Terraform project directory in Cloud Shell.
   - Run `terraform init` and `terraform apply`. Terraform will utilize your user account's credentials for authentication.

   **Recommendation:** If you're using this option, it's highly recommended to follow the principle of least privilege:

   - **Assign Permissions Temporarily:** Grant your user account the required roles only during the deployment process. Revoke the roles immediately after the deployment is complete.
   - **Use `gcloud config set auth/impersonate_service_account`:** If possible, impersonate a service account with limited permissions even within your Cloud Shell session, similar to Option 1.

**Important Security Considerations:**

- **Principle of Least Privilege:** Always grant only the minimum necessary permissions to service accounts and user accounts involved in your Terraform deployments. Regularly review and revoke unnecessary permissions.
- **Secure Secret Management:** Use [Google Cloud Secret Manager](https://cloud.google.com/secret-manager) to store sensitive information like API keys and passwords referenced in your Terraform code.
- **Follow Google Cloud's Security Best Practices:** Refer to the [security checklist](https://cloud.google.com/docs/enterprise/setup-checklist) and other security documentation for comprehensive guidance on securing your GCP environment.
- **Terraform State Management:** Store your [Terraform state](https://cloud.google.com/docs/terraform/best-practices#manage-terraform-state) remotely in a secure location like Google Cloud Storage and enable encryption at rest. 



## Next Steps
1. Return to the Cloud Setup experience to set up your team-Level [identity and access management.](https://console.cloud.google.com/cloud-setup/workloads)
2. Deploy your workloads on top of this foundation.
    - See [Solutions Center](https://solutions.cloud.google.com/home).
    - See [SMB hero packages](https://docs.google.com/presentation/d/1Ef9Jn6r8N_9EZGs74M5VN-_1fUZz30BbR3ge3j8Cf8o/present?slide=id.g2b6b826f79b_0_5).
    - Migrate workloads using the networking you set up in this foundation.
    - See the [Migrations Center](https://cloud.google.com/migration-center/docs/migration-center-overview).
3. Use GitOps and CICD to manage deployments  [Learn more](https://cloud.google.com/docs/terraform/resource-management/managing-infrastructure-as-code).






# Appendix

## Bootstrap 

### If you did not configure a billing account admin during the initial setup, you might need to manually create the following resources:

1. **Terraform Host Project and Google Cloud Storage (GCS) Bucket:**
A dedicated Google Cloud Project is recommended to host your Terraform state files and run Terraform deployments. Additionally, a GCS bucket within this project will store the state files, enabling collaboration and version control.

   ```bash
   # Create the Terraform host project
   PROJECT_ID="your-terraform-host-project"
   gcloud projects create $PROJECT_ID

   # Enable required APIs for the project
   gcloud services enable cloudresourcemanager.googleapis.com storage-api.googleapis.com storage.googleapis.com --project=$PROJECT_ID

   # Create a Cloud Storage bucket to store Terraform state
   BUCKET_NAME="your-terraform-state-bucket"
   gsutil mb -p $PROJECT_ID -l US gs://$BUCKET_NAME

   #  Configure versioning the bucket for Terraform state
   gsutil versioning set on gs://$BUCKET_NAME
   ```

   Replace <bucketname> with a bucket name of your configured in your backends.tf. Location can be changed to one of the options [listed here](https://cloud.google.com/storage/docs/locations)
   

2. **Terraform Service Account**
Service account to access deploy terraform
   ```bash
   # Create service account
   gcloud iam service-accounts create terraform-deployer --display-name "Terraform Deployer"

   # Assign permissions to terraform bucket
   gsutil iam ch role roles/storage.objectAdmin serviceAccount:terraform-deployer@<PROJECT_ID>.iam.gserviceaccount.com gs://<bucketname>
   ```

## Configure a Git Repository

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

4. **Create a Git repository and push your local repository:**
   ```bash
   git remote add origin <your-git-repository-url>
   git push -u origin main
   ```

5. **Clone the Repository (optional):** Verify by cloning the repository to your local machine.
  ```bash
  git clone <your-github-repository-url>
  cd <your-github-repository>
  ```


# Infrastructure as Code with Terraform — Exercises & Solutions

**Repository to use:** [GitLab – terraform-exercises](https://gitlab.com/twn-devops-bootcamp/latest/12-terraform/terraform-exercises)

Your K8s cluster on AWS is successfully running and used as a production environment. Your team wants to have additional K8s environments for development, test and staging with the same exact configuration and setup, so they can properly test and try out new features before releasing to production. So you must create 3 more EKS clusters.

But you don't want to do that manually 3 times, so you decide it would be much more efficient to script creating the EKS cluster and execute that same script 3 times to create 3 more identical environments.

---

## Exercise 1: Create Terraform project to spin up EKS cluster & use MySQL for data persistence

**Task:**
Create a Terraform project that spins up an EKS cluster with the exact same setup that you created in the previous exercise, for the same Java Gradle application:
- Create EKS cluster with 3 Nodes and 1 Fargate profile only for your java application
- Deploy Mysql with 3 replicas with volumes for data persistence using helm

Create a separate git repository for your Terraform project, separate from the Java application, so that changes to the EKS cluster can be made by a separate team independent of the application changes themselves.

**Solution:**

This project provisions an EKS cluster with the following configuration:

⚠️ **Important:** Make sure to follow the instructions outlined in the README file to configure K8s storage correctly!

**Configuration Details:**
- **S3 bucket** as a storage for Terraform state
- K8s cluster with **3 nodes** and **1 fargate profile** for "my-app" namespace with **EBS CSI Driver add-on installed and correct node group permissions configured**
- **Mysql** chart with 3 replicas
- K8s version **1.28**
- AWS region for VPC, EKS and S3 bucket: **"eu-north-1"**

⚠️ **Important:** Make sure to change the region for your cluster in all relevant places!

ℹ️ Check **README.md** file for the exact versions used in the projects for:
- *Terraform*
- *Terraform modules* 
- *Terraform providers*

**To execute the project:**
```sh
# set variables values in the "dev.tfvars" file
# set "bucket name" and "bucket region" values in the terraform configuration in the "vpc.tf" file

# installs all the providers and modules used in the project
terraform init

# executes the Terraform script
terraform apply
```

**To access the cluster with kubectl, once it's configured:**
```sh
# configure kubeconfig
aws eks update-kubeconfig --name {cluster-name} --region {your-region}

# example:
aws eks update-kubeconfig --name my-cluster --region eu-north-1
```

ℹ️ This will configure the kubeconfig file in the ~/.kube/ folder

**To verify the cluster access:**
```sh
kubectl get nodes
eksctl get fargateprofile --cluster my-cluster
```

**Tips:** 
- Ensure you add the AWS EBS CSI Driver to your project before creating the resources - additional information can be found in the README file of the project
- Also ensure your MySQL configuration is configured to be dependent on the EKS cluster to avoid any errors: `depends_on = [module.eks.cluster_name]`

---

## Exercise 2: Configure remote state

**Task:**
By default, TF stores state locally. You know that this is not practical when working in a team, because each user must make sure they always have the latest state data before running Terraform. To fix that, you:
- Configure remote state with a remote data store for your terraform project

You can use an S3 bucket for storage.

Now, the platform team that manages K8s clusters want to make changes to the cluster configurations based on the Infrastructure as Code best practices: They collaborate and commit changes to a git repository and those changes get applied to the cluster through a CI/CD pipeline.

So the AWS infrastructure and K8s cluster changes will be deployed the same way as the application changes, using a CI/CD pipeline.

So the team asks you to help them create a separate Jenkins pipeline for the Terraform project, in addition to your java-app pipeline from the previous module.

**Solution:**

The remote state configuration is included in the Terraform project setup from Exercise 1. The S3 bucket configuration for storing Terraform state is already implemented in the project structure.

**Configuration includes:**
- S3 bucket as remote backend for Terraform state
- Proper bucket configuration in `vpc.tf` file
- State locking capabilities for team collaboration

---

## Exercise 3: CI/CD pipeline for Terraform project

**Task:**
- Create a separate Jenkins pipeline for Terraform provisioning the EKS cluster

**Solution:**

This project includes a Jenkinsfile for CI/CD pipeline.

**Environment Variables Configuration:**

Values of the following environment variables need to be set inside Jenkinsfile:
```jenkins
TF_VAR_env_prefix = "dev"
TF_VAR_k8s_version = "1.28"
TF_VAR_cluster_name = "my-cluster"
TF_VAR_region = "eu-north-1"
```

**Jenkins Credentials Configuration:**

Values of the following environment variables need to be configured as Jenkins credentials:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

**Pipeline Setup:**
1. Create a new Jenkins pipeline job
2. Configure the pipeline to use the Jenkinsfile from your Terraform repository
3. Set up the required credentials in Jenkins
4. Configure the environment variables as specified above
5. Run the pipeline to provision your EKS cluster infrastructure

The pipeline will handle:
- Terraform initialization
- Planning and validation
- Infrastructure provisioning
- State management with remote backend

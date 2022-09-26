# GCP3Tier
# What is Terraform?
Terrraform is an open-source Infrastructure as Code (IaC) tool developed by HashiCorp. 
It is used to define and provision a complete infrastructure using a declarative language.
IaC helps businesses automate their infrastructures by programmatically managing an entire technology stack through code.

# Terraform core concepts:
Variables: Also used as input-variables, it is key-value pair used by Terraform modules to allow customization.

Provider: It is a plugin to interact with APIs of service and access its related resources. (We will be using AWS for this project)

Module: It is a folder with Terraform templates where all the configurations are defined.

Resources: refers to a block of one or more infrastructure objects (compute instances, virtual networks, etc.), which are used in configuring and managing the infrastructure.

Output Values: These are return values of a terraform module that can be used by other configurations.

Plan: It is one stage where it determines what needs to be created, updated, or destroyed.

Apply: It is the last stage where it applies the changes of the infrastructure in order to move to the desired state.

# Build terraform module for a Three-Tier application on GCP:

Enable services to start provisioning environment

```
!/bin/bash
gcloud services enable compute.googleapis.com
gcloud services enable dns.googleapis.com
gcloud services enable storage-api.googleapis.com
gcloud services enable container.googleapis.com
```

Project was created with a random id.

# Step1


# Build a Global vpc with automatic creation of subnets:
```
resource "google_compute_network" "vpc-network-team3" {
  name                    = var.vpc_name
  auto_create_subnetworks = "true"
  routing_mode            = "GLOBAL"
}
```
## Module was released in Terraform registry.

# Step2

On top of VPC from previous step, please create Auto Scaling Group. Auto Scaling Group should use minimum 1 instance and should create it's Load Balancer.

### This block of code adds an autoscaling group in a zone specified in the variables file using an instance group manager as a target and is dependent on SQL.
```
resource "google_compute_autoscaler" "team3" {
     depends_on = [
        google_sql_database_instance.database,
        
    ]
  name   = var.ASG_name
  zone   = var.zone
  target = google_compute_instance_group_manager.my-igm.self_link
```
# Created Load Balancer
```
module "lb" {
  source       = "GoogleCloudPlatform/lb/google"
  version      = "2.2.0"
  region       = var.region
  name         = var.lb_name
  service_port = 80
  target_tags  = ["my-target-pool"]
  network      = google_compute_network.vpc-network-team3.name
}
```

# Step3

Create CloudSQL to use with wordpress autoscaling group

# Create a DB instance with a DB inside it and create a user
```
resource "google_sql_database_instance" "database" {
  name                = var.dbinstance_name
  database_version    = var.data_base_version
  region              = var.region
  root_password       = var.db_password
  deletion_protection = "false"
  project             = var.project_name



  settings {
    tier = "db-f1-micro"

ip_configuration {
      ipv4_enabled = "true"

      authorized_networks {
        value           = var.authorized_networks
        name            = var.db_username
        
      }
resource "google_sql_database" "database" {
  name     = var.db_name
  instance = google_sql_database_instance.database.name
}

resource "google_sql_user" "users" {
  name     = var.db_username
  instance = google_sql_database_instance.database.name
  host     = var.db_host
  password = var.db_password
}

```
# Makefile - create or destroy in different regions

```
virginia:
		terraform workspace new virginia || terraform workspace select virginia
		terraform init
		terraform apply -var-file envs/virginia.tfvars -auto-approve
iowa:
		terraform workspace new iowa ||terraform workspace select iowa
		terraform init
		terraform apply -var-file envs/iowa.tfvars -auto-approve
losangeles:
		terraform workspace new losangeles || terraform workspace select losangeles
		terraform init
		terraform apply -var-file envs/losangeles.tfvars -auto-approve
lasvegas:
		terraform workspace new lasvegas || terraform workspace select lasvegas
		terraform init
		terraform apply -var-file envs/lasvegas.tfvars -auto-approve		

################################################################################################

destroy-virginia:
		terraform workspace new virginia || terraform workspace select virginia
		terraform init
		terraform destroy -var-file envs/virginia.tfvars -auto-approve
destroy-iowa:
		terraform workspace new iowa ||terraform workspace select iowa
		terraform destroy -var-file envs/iowa.tfvars -auto-approve
losangeles:
		terraform workspace new losangeles || terraform workspace select losangeles
		terraform destroy -var-file envs/losangeles.tfvars -auto-approve
lasvegas:
		terraform workspace new lasvegas || terraform workspace select lasvegas
		terraform destroy -var-file envs/lasvegas.tfvars -auto-approve
```

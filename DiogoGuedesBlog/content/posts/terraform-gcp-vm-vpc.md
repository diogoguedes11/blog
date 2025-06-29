---
title: "Deploy Your First VM and VPC in Google Cloud with Terraform"
date: 2025-06-29T15:00:00+00:00
tags: ["terraform", "gcp", "google-cloud", "infrastructure-as-code", "vpc", "virtual-machine", "devops"]
author: "Diogo Guedes"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Complete guide to deploying your first virtual machine and VPC in Google Cloud Platform using Terraform infrastructure as code"
disableHLJS: false
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
cover:
    image: ""
    alt: "Terraform and Google Cloud Platform"
    caption: "Learn how to deploy GCP infrastructure with Terraform"
    relative: false
    hidden: false
---

Google Cloud Platform (GCP) is one of the leading cloud providers, and Terraform is the de facto standard for Infrastructure as Code (IaC). In this comprehensive guide, you'll learn how to deploy your first virtual machine and VPC in GCP using Terraform.

## Prerequisites

Before we begin, make sure you have:

- **Google Cloud Account** with billing enabled
- **Terraform** installed (v1.0+)
- **Google Cloud SDK** (`gcloud`) installed and configured
- **Basic understanding** of cloud networking concepts

## Setting Up Your Environment

### 1. Install Required Tools

```bash
# Install Terraform (macOS with Homebrew)
brew install terraform

# Install Google Cloud SDK
brew install --cask google-cloud-sdk

# Verify installations
terraform version
gcloud version
```

### 2. Authenticate with Google Cloud

```bash
# Login to your Google Cloud account
gcloud auth login

# Set your default project
gcloud config set project YOUR_PROJECT_ID

# Enable required APIs
gcloud services enable compute.googleapis.com
gcloud services enable servicenetworking.googleapis.com
```

### 3. Create Service Account for Terraform

```bash
# Create service account
gcloud iam service-accounts create terraform-sa \
    --display-name="Terraform Service Account"

# Grant necessary permissions
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
    --member="serviceAccount:terraform-sa@YOUR_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/compute.admin"

gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
    --member="serviceAccount:terraform-sa@YOUR_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/iam.serviceAccountUser"

# Create and download key
gcloud iam service-accounts keys create terraform-key.json \
    --iam-account=terraform-sa@YOUR_PROJECT_ID.iam.gserviceaccount.com
```

## Project Structure

Let's organize our Terraform code properly:

```
terraform-gcp-vm/
├── main.tf              # Main configuration
├── variables.tf         # Input variables
├── outputs.tf           # Output values
├── terraform.tfvars     # Variable values
└── terraform-key.json  # Service account key
```

## Creating the Terraform Configuration

### 1. Provider Configuration (`main.tf`)

```hcl
terraform {
  required_version = ">= 1.0"
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
}

provider "google" {
  credentials = file("terraform-key.json")
  project     = var.project_id
  region      = var.region
  zone        = var.zone
}

# Create VPC Network
resource "google_compute_network" "vpc_network" {
  name                    = var.network_name
  auto_create_subnetworks = false
  description             = "Custom VPC network for VM deployment"
}

# Create Subnet
resource "google_compute_subnetwork" "subnet" {
  name          = var.subnet_name
  ip_cidr_range = var.subnet_cidr
  region        = var.region
  network       = google_compute_network.vpc_network.id
  description   = "Subnet for VM instances"

  # Enable private Google access
  private_ip_google_access = true
}

# Create Firewall Rules
resource "google_compute_firewall" "allow_ssh" {
  name    = "${var.network_name}-allow-ssh"
  network = google_compute_network.vpc_network.name

  allow {
    protocol = "tcp"
    ports    = ["22"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["ssh-allowed"]
  description   = "Allow SSH access from anywhere"
}

resource "google_compute_firewall" "allow_http" {
  name    = "${var.network_name}-allow-http"
  network = google_compute_network.vpc_network.name

  allow {
    protocol = "tcp"
    ports    = ["80", "443"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["web-server"]
  description   = "Allow HTTP and HTTPS traffic"
}

resource "google_compute_firewall" "allow_internal" {
  name    = "${var.network_name}-allow-internal"
  network = google_compute_network.vpc_network.name

  allow {
    protocol = "tcp"
    ports    = ["0-65535"]
  }

  allow {
    protocol = "udp"
    ports    = ["0-65535"]
  }

  allow {
    protocol = "icmp"
  }

  source_ranges = [var.subnet_cidr]
  description   = "Allow internal communication within the subnet"
}

# Create External IP
resource "google_compute_address" "static_ip" {
  name         = "${var.vm_name}-static-ip"
  region       = var.region
  description  = "Static external IP for VM"
}

# Create VM Instance
resource "google_compute_instance" "vm_instance" {
  name         = var.vm_name
  machine_type = var.machine_type
  zone         = var.zone

  tags = ["ssh-allowed", "web-server"]

  boot_disk {
    initialize_params {
      image = var.vm_image
      size  = var.disk_size
      type  = "pd-standard"
    }
  }

  network_interface {
    network    = google_compute_network.vpc_network.id
    subnetwork = google_compute_subnetwork.subnet.id
    
    access_config {
      nat_ip = google_compute_address.static_ip.address
    }
  }

  metadata = {
    ssh-keys = "${var.ssh_username}:${file(var.ssh_public_key_path)}"
  }

  metadata_startup_script = <<-EOF
    #!/bin/bash
    apt-get update
    apt-get install -y nginx
    systemctl start nginx
    systemctl enable nginx
    echo "<h1>Hello from ${var.vm_name}!</h1>" > /var/www/html/index.html
    echo "<p>Deployed with Terraform on GCP</p>" >> /var/www/html/index.html
  EOF

  service_account {
    email  = google_service_account.vm_service_account.email
    scopes = ["cloud-platform"]
  }

  labels = {
    environment = var.environment
    managed_by  = "terraform"
  }
}

# Create Service Account for VM
resource "google_service_account" "vm_service_account" {
  account_id   = "${var.vm_name}-sa"
  display_name = "Service Account for ${var.vm_name}"
  description  = "Service account for VM instance"
}

# IAM binding for VM service account
resource "google_project_iam_member" "vm_service_account_binding" {
  project = var.project_id
  role    = "roles/compute.osLogin"
  member  = "serviceAccount:${google_service_account.vm_service_account.email}"
}
```

### 2. Variables (`variables.tf`)

```hcl
variable "project_id" {
  description = "GCP Project ID"
  type        = string
}

variable "region" {
  description = "GCP Region"
  type        = string
  default     = "us-central1"
}

variable "zone" {
  description = "GCP Zone"
  type        = string
  default     = "us-central1-a"
}

variable "network_name" {
  description = "Name of the VPC network"
  type        = string
  default     = "my-vpc-network"
}

variable "subnet_name" {
  description = "Name of the subnet"
  type        = string
  default     = "my-subnet"
}

variable "subnet_cidr" {
  description = "CIDR range for the subnet"
  type        = string
  default     = "10.0.1.0/24"
}

variable "vm_name" {
  description = "Name of the VM instance"
  type        = string
  default     = "my-vm-instance"
}

variable "machine_type" {
  description = "Machine type for the VM"
  type        = string
  default     = "e2-micro"
}

variable "vm_image" {
  description = "Image for the VM"
  type        = string
  default     = "ubuntu-os-cloud/ubuntu-2204-lts"
}

variable "disk_size" {
  description = "Size of the boot disk in GB"
  type        = number
  default     = 20
}

variable "ssh_username" {
  description = "SSH username"
  type        = string
  default     = "terraform-user"
}

variable "ssh_public_key_path" {
  description = "Path to SSH public key"
  type        = string
  default     = "~/.ssh/id_rsa.pub"
}

variable "environment" {
  description = "Environment tag"
  type        = string
  default     = "development"
}
```

### 3. Outputs (`outputs.tf`)

```hcl
output "vpc_network_name" {
  description = "Name of the VPC network"
  value       = google_compute_network.vpc_network.name
}

output "vpc_network_id" {
  description = "ID of the VPC network"
  value       = google_compute_network.vpc_network.id
}

output "subnet_name" {
  description = "Name of the subnet"
  value       = google_compute_subnetwork.subnet.name
}

output "subnet_cidr" {
  description = "CIDR range of the subnet"
  value       = google_compute_subnetwork.subnet.ip_cidr_range
}

output "vm_name" {
  description = "Name of the VM instance"
  value       = google_compute_instance.vm_instance.name
}

output "vm_internal_ip" {
  description = "Internal IP address of the VM"
  value       = google_compute_instance.vm_instance.network_interface[0].network_ip
}

output "vm_external_ip" {
  description = "External IP address of the VM"
  value       = google_compute_address.static_ip.address
}

output "ssh_connection_command" {
  description = "SSH command to connect to the VM"
  value       = "ssh ${var.ssh_username}@${google_compute_address.static_ip.address}"
}

output "web_url" {
  description = "URL to access the web server"
  value       = "http://${google_compute_address.static_ip.address}"
}
```

### 4. Variable Values (`terraform.tfvars`)

```hcl
project_id           = "your-gcp-project-id"
region              = "us-central1"
zone                = "us-central1-a"
network_name        = "terraform-vpc"
subnet_name         = "terraform-subnet"
subnet_cidr         = "10.0.1.0/24"
vm_name             = "terraform-vm"
machine_type        = "e2-micro"
ssh_username        = "terraform-user"
ssh_public_key_path = "~/.ssh/id_rsa.pub"
environment         = "development"
```

## Deploying the Infrastructure

### 1. Generate SSH Key Pair

```bash
# Generate SSH key pair if you don't have one
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
```

### 2. Initialize and Deploy

```bash
# Initialize Terraform
terraform init

# Validate configuration
terraform validate

# Plan the deployment
terraform plan

# Apply the configuration
terraform apply
```

### 3. Verify Deployment

```bash
# Check the outputs
terraform output

# SSH into the VM
ssh terraform-user@$(terraform output -raw vm_external_ip)

# Test the web server
curl http://$(terraform output -raw vm_external_ip)
```

## Best Practices

### 1. Security Considerations

```hcl
# Use more restrictive firewall rules
resource "google_compute_firewall" "allow_ssh_restricted" {
  name    = "${var.network_name}-allow-ssh-restricted"
  network = google_compute_network.vpc_network.name

  allow {
    protocol = "tcp"
    ports    = ["22"]
  }

  # Restrict to your IP address
  source_ranges = ["YOUR_IP_ADDRESS/32"]
  target_tags   = ["ssh-allowed"]
}
```

### 2. State Management

```hcl
terraform {
  backend "gcs" {
    bucket = "your-terraform-state-bucket"
    prefix = "terraform/state"
  }
}
```

### 3. Resource Tagging

```hcl
locals {
  common_labels = {
    environment = var.environment
    project     = "terraform-gcp-demo"
    managed_by  = "terraform"
    created_by  = "diogo-guedes"
  }
}

resource "google_compute_instance" "vm_instance" {
  # ... other configuration ...
  
  labels = local.common_labels
}
```

## Advanced Configurations

### Multiple VMs with Load Balancer

```hcl
# Instance Template
resource "google_compute_instance_template" "web_template" {
  name_prefix  = "web-template-"
  machine_type = var.machine_type

  disk {
    source_image = var.vm_image
    auto_delete  = true
    boot         = true
  }

  network_interface {
    network    = google_compute_network.vpc_network.id
    subnetwork = google_compute_subnetwork.subnet.id
  }

  metadata_startup_script = file("startup-script.sh")

  lifecycle {
    create_before_destroy = true
  }
}

# Managed Instance Group
resource "google_compute_instance_group_manager" "web_igm" {
  name               = "web-igm"
  base_instance_name = "web"
  zone               = var.zone
  target_size        = 2

  version {
    instance_template = google_compute_instance_template.web_template.id
  }
}
```

## Cleanup

When you're done testing, clean up the resources:

```bash
# Destroy all resources
terraform destroy

# Confirm by typing 'yes'
```

## Troubleshooting

### Common Issues

1. **Authentication Errors**
   ```bash
   # Re-authenticate
   gcloud auth application-default login
   ```

2. **API Not Enabled**
   ```bash
   # Enable required APIs
   gcloud services enable compute.googleapis.com
   ```

3. **Insufficient Permissions**
   ```bash
   # Check your permissions
   gcloud projects get-iam-policy YOUR_PROJECT_ID
   ```

## Cost Optimization

- Use **e2-micro** instances for testing (eligible for free tier)
- Set up **budget alerts** in GCP Console
- Use **preemptible instances** for non-critical workloads
- Implement **auto-shutdown** for development environments

## Conclusion

You've successfully deployed your first virtual machine and VPC in Google Cloud Platform using Terraform! This foundation gives you:

- **Reproducible Infrastructure** - Deploy identical environments
- **Version Control** - Track infrastructure changes
- **Collaboration** - Share infrastructure code with your team
- **Scalability** - Easily add more resources

## Next Steps

- Explore **Terraform modules** for reusable components
- Implement **CI/CD pipelines** for infrastructure deployment
- Learn about **GCP networking** features like Cloud NAT and VPN
- Set up **monitoring and logging** with Cloud Operations

## Resources

- [Terraform GCP Provider Documentation](https://registry.terraform.io/providers/hashicorp/google/latest/docs)
- [Google Cloud Architecture Center](https://cloud.google.com/architecture)
- [Terraform Best Practices](https://www.terraform.io/docs/cloud/guides/recommended-practices/index.html)
- [GCP Free Tier](https://cloud.google.com/free)

Happy terraforming! 🚀

---

*Have questions or suggestions? Feel free to reach out on [LinkedIn](https://www.linkedin.com/in/diogo-guedes11/) or check out more DevOps content on this blog!*

# Top 10 Essential GCP Services with Terraform Examples

## Table of Contents

1. [Compute Engine](#1-compute-engine)
2. [Cloud Storage](#2-cloud-storage)
3. [Cloud SQL](#3-cloud-sql)
4. [Cloud Functions](#4-cloud-functions)
5. [Virtual Private Cloud (VPC)](#5-virtual-private-cloud)
6. [Cloud IAM](#6-cloud-iam)
7. [Google Kubernetes Engine (GKE)](#7-google-kubernetes-engine)
8. [Cloud Run](#8-cloud-run)
9. [Cloud Pub/Sub](#9-cloud-pubsub)
10. [Cloud Load Balancing](#10-cloud-load-balancing)

## Prerequisites

- Google Cloud Platform (GCP) account
- Google Cloud SDK installed and configured
- Terraform installed

## 1. Compute Engine {#1-compute-engine}

### Overview of Compute Engine

Scalable, high-performance virtual machines running in Google's data centers.

### Compute Engine Terraform Example

```hcl
provider "google" {
  project = "your-project-id"
  region  = "us-central1"
}

resource "google_compute_instance" "default" {
  name         = "example-instance"
  machine_type = "e2-micro"
  zone         = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
    network = "default"
    access_config {
      // Ephemeral IP
    }
  }
}
```

## 2. Cloud Storage {#2-cloud-storage}

### Overview of Cloud Storage

Unified object storage for developers and enterprises.

### Cloud Storage Terraform Example

```hcl
resource "google_storage_bucket" "static-site" {
  name          = "example-bucket-name"
  location      = "US"
  force_destroy = true

  uniform_bucket_level_access = true
}
```

## 3. Cloud SQL {#3-cloud-sql}

### Overview of Cloud SQL

Fully-managed database service for MySQL, PostgreSQL, and SQL Server.

### Cloud SQL Terraform Example

```hcl
resource "google_sql_database_instance" "instance" {
  name             = "example-instance"
  database_version = "POSTGRES_13"
  region           = "us-central1"

  settings {
    tier = "db-f1-micro"
  }

}

resource "google_sql_database" "database" {
  name     = "example-db"
  instance = google_sql_database_instance.instance.name
}
```

## 4. Cloud Functions {#4-cloud-functions}

### Overview of Cloud Functions

Serverless execution environment for building and connecting cloud services.

### Cloud Functions Terraform Example

```hcl
resource "google_storage_bucket" "bucket" {
  name = "example-function-bucket"
}

resource "google_storage_bucket_object" "archive" {
  name   = "function-source.zip"
  bucket = google_storage_bucket.bucket.name
  source = "function-source.zip"
}

resource "google_cloudfunctions_function" "function" {
  name        = "example-function"
  description = "My Cloud Function"
  runtime     = "nodejs14"

  available_memory_mb   = 128
  source_archive_bucket = google_storage_bucket.bucket.name
  source_archive_object = google_storage_bucket_object.archive.name
  trigger_http          = true
  entry_point           = "helloWorld"
}
```

## 5. Virtual Private Cloud (VPC) {#5-virtual-private-cloud}

### Overview of VPC

Virtual networking functionality for GCP resources.

### VPC Terraform Example

```hcl
resource "google_compute_network" "vpc_network" {
  name                    = "example-vpc"
  auto_create_subnetworks = false
  mtu                     = 1460
}

resource "google_compute_subnetwork" "subnet" {
  name          = "example-subnet"
  ip_cidr_range = "10.2.0.0/16"
  region        = "us-central1"
  network       = google_compute_network.vpc_network.id
}
```

## Next Steps

1. Run `gcloud auth application-default login` to authenticate
2. Run `terraform init` to initialize the provider
3. Run `terraform plan` to see the execution plan
4. Run `terraform apply` to create the resources

## Best Practices

- Use service accounts for application authentication
- Implement VPC Service Controls for sensitive data
- Enable audit logging for all services
- Use Cloud IAM for fine-grained access control
- Implement organization policies for compliance

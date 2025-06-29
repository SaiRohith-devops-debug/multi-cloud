# Top 10 Essential AWS Services with Terraform Examples

## Table of Contents

1. [EC2 (Elastic Compute Cloud)](#ec2)
2. [S3 (Simple Storage Service)](#s3)
3. [RDS (Relational Database Service)](#rds)
4. [Lambda](#lambda)
5. [VPC (Virtual Private Cloud)](#vpc)
6. [IAM (Identity and Access Management)](#iam)
7. [CloudFront](#cloudfront)
8. [API Gateway](#api-gateway)
9. [DynamoDB](#dynamodb)
10. [SNS (Simple Notification Service)](#sns)

## Prerequisites

- AWS Account with appropriate permissions
- Terraform installed
- AWS CLI configured with credentials

## <a name="ec2"></a>1. EC2 (Elastic Compute Cloud)

### Overview

EC2 provides resizable compute capacity in the cloud.

### Terraform Example

```hcl
# main.tf
provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"  # Amazon Linux 2 AMI
  instance_type = "t2.micro"
  key_name      = "my-key-pair"

  tags = {
    Name = "web-server"
  }
}
```

## <a name="s3"></a>2. S3 (Simple Storage Service)

### Overview

S3 provides object storage through a web service interface.

### Terraform Example

```hcl
resource "aws_s3_bucket" "example" {
  bucket = "my-unique-bucket-name-12345"
  acl    = "private"

  versioning {
    enabled = true
  }

  tags = {
    Environment = "Dev"
  }
}
```

## <a name="vpc"></a>3. VPC (Virtual Private Cloud)

### Overview

VPC lets you provision a logically isolated section of the AWS Cloud.

### Terraform Example

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  
  tags = {
    Name = "main-vpc"
  }
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
  
  tags = {
    Name = "public-subnet"
  }
}
```

## <a name="rds"></a>4. RDS (Relational Database Service)

### Overview

RDS makes it easy to set up, operate, and scale a relational database.

### Terraform Example

```hcl
resource "aws_db_instance" "default" {
  allocated_storage    = 20
  storage_type         = "gp2"
  engine               = "mysql"
  engine_version       = "5.7"
  instance_class       = "db.t2.micro"
  name                 = "mydb"
  username             = "admin"
  password             = "yourpassword123"
  parameter_group_name = "default.mysql5.7"
  skip_final_snapshot  = true
}
```

## <a name="lambda"></a>5. Lambda

### Overview

AWS Lambda lets you run code without provisioning or managing servers.

### Terraform Example

```hcl
resource "aws_lambda_function" "test_lambda" {
  filename      = "lambda_function_payload.zip"
  function_name = "lambda_function_name"
  role          = aws_iam_role.iam_for_lambda.arn
  handler       = "index.handler"
  runtime       = "nodejs14.x"
}
```

## <a name="iam"></a>6. IAM (Identity and Access Management)

### Overview

IAM enables you to manage access to AWS services and resources securely.

### Terraform Example

```hcl
resource "aws_iam_role" "example" {
  name = "example-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Sid    = ""
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      },
    ]
  })
}
```

## <a name="auto-scaling"></a>7. Auto Scaling

### Overview

Auto Scaling helps maintain application availability.

### Terraform Example

```hcl
resource "aws_launch_configuration" "example" {
  image_id        = "ami-0c55b159cbfafe1f0"
  instance_type   = "t2.micro"
  security_groups = [aws_security_group.instance.id]

  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World" > index.html
              nohup busybox httpd -f -p 8080 &
              EOF
}

resource "aws_autoscaling_group" "example" {
  launch_configuration = aws_launch_configuration.example.name
  vpc_zone_identifier = [aws_subnet.public.id]

  min_size = 2
  max_size = 10

  tag {
    key                 = "Name"
    value               = "terraform-asg-example"
    propagate_at_launch = true
  }
}
```

## <a name="route53"></a>8. Route 53

### Overview

Route 53 is a scalable and highly available DNS service.

### Terraform Example

```hcl
resource "aws_route53_zone" "primary" {
  name = "example.com"
}

resource "aws_route53_record" "www" {
  zone_id = aws_route53_zone.primary.zone_id
  name    = "www.example.com"
  type    = "A"
  ttl     = 300
  records = [aws_eip.lb.public_ip]
}
```

## <a name="cloudfront"></a>9. CloudFront

### Overview

Amazon CloudFront is a fast content delivery network (CDN) service.

### Terraform Example

```hcl
resource "aws_cloudfront_distribution" "s3_distribution" {
  origin {
    domain_name = aws_s3_bucket.example.bucket_regional_domain_name
    origin_id   = "S3-${aws_s3_bucket.example.bucket}"
  }


  enabled             = true
  default_root_object = "index.html"

  default_cache_behavior {
    allowed_methods  = ["DELETE", "GET", "HEAD", "OPTIONS", "PATCH", "POST", "PUT"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "S3-${aws_s3_bucket.example.bucket}"

    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }

    viewer_protocol_policy = "allow-all"
    min_ttl                = 0
    default_ttl            = 3600
    max_ttl                = 86400
  }
}
```

## <a name="eks"></a>10. EKS (Elastic Kubernetes Service)

### Overview

Amazon EKS gives you the flexibility to start, run, and scale Kubernetes applications.

### Terraform Example

```hcl
module "eks" {
  source          = "terraform-aws-modules/eks/aws"
  cluster_name    = "my-cluster"
  cluster_version = "1.21"
  subnets         = ["subnet-abcde012", "subnet-bcde012a", "subnet-fghi345a"]
  vpc_id          = "vpc-1234556abcdef"

  worker_groups = [
    {
      instance_type = "t2.micro"
      asg_max_size  = 3
    }
  ]
}
```

## Next Steps

1. Run `terraform init` to initialize the provider
2. Run `terraform plan` to see the execution plan
3. Run `terraform apply` to create the resources
4. Use `terraform destroy` to clean up resources when done

## Best Practices

- Use variables for sensitive information
- Implement remote state storage
- Use workspaces for environment separation
- Implement proper tagging strategy
- Use modules for reusable components

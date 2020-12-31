# cloud6
# Deploy the Wordpress application on Kubernetes and AWS using terraform

## Problem Statement:
Deploy the Wordpress application on Kubernetes and AWS using terraform including the following steps:-

- Infrastructure as code using terraform, which automatically deploy the Wordpress application
- On AWS, use RDS service for the relational database for Wordpress application.
- Deploy the Wordpress as a container either on top of Minikube 
- The Wordpress application should be accessible from the public world deployed on workstation if deployed on Minikube.

## Prerequisites:
- AWS CLIv2 installed and configured with IAM user.
- Minikube, Terraform installed.
- kubectl configured.

# Implementation:

## Step 1: On AWS, use RDS service for the relational database for Wordpress application:

```html
#Declare provider
provider "aws" {
  region = "ap-south-1"
}


#Create AWS security group for AWS RDS
resource "aws_security_group" "project6_mysql_rds" {
  name        = "RDS-Security-Group"
  description = "MySQL Ports"
 
  ingress {
    description = "Mysql_RDS"
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
tags = {
    Name = "Project6_Mysql-RDS"
  }
}

#Create AWS Relational Database instance on AWS 
resource "aws_db_instance" "default" {
  allocated_storage    = 20   // 20 GB allocate 
  storage_type         = "gp2"  //storage parameter
  engine               = "mysql"  //mysql engine used 
  engine_version       = "5.7"  // mysql version
  instance_class       = "db.t2.micro"  //specification of instance
  name                 = "mywpdb"  //Name of this resource
  username             = "ashu"  //username
  password             = "ashu123456789"  //password
  parameter_group_name = "default.mysql5.7"
  publicly_accessible = true  //Grant access to public world
  skip_final_snapshot = true
  vpc_security_group_ids = [aws_security_group.project6_mysql_rds.id]
  tags = {
  name = "MyRDS"
   }
}

output "dbdns" {
    value = aws_db_instance.default.address
} 
```

## Step 2: Infrastructure as code using terraform, which automatically deploy the Wordpress application:

```html
#Deploying Frontend application on kubernetes
provider "kubernetes" {
  config_context_cluster   = "minikube"
}
resource "kubernetes_deployment" "mywp" {
  metadata {
    name = "wordpress"
  }
    
  spec {
    replicas = 1
    selector {
          match_labels = {
              App = "wordpress"
              region = "IN"
              env = "frontend"
          }
          match_expressions {
              key = "env"
              operator = "In"
              values = ["frontend","webserver"]

          }
    } 

    template {
        metadata {
          labels = {
            App = "wordpress"
            region = "IN"
            env = "frontend"
        }
      }
    spec {
        container {
          image = "wordpress:5.1.1-php7.3-apache"
          name  = "mywpfrontend"
        }
      }
    }
  }
}
```

## Step 3: Exposing the Wordpress as a container on top of Minikube 

```html
resource "kubernetes_service" "mywp-service"{
  depends_on = [kubernetes_deployment.mywp]
  metadata {
    name = "mywp-service"
  }
  spec {
    selector = {
      App = kubernetes_deployment.mywp.spec.0.template.0.metadata[0].labels.App
    }
    port {
      node_port = 30001
      port        = 80
      target_port = 80
    }
    type = "NodePort"
  }
}
```

## Step 4: Run the following commands
``` html
- Initialize the workspace 

terraform init

- Apply Terraform.

terraform apply -auto-approve

```

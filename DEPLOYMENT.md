# ArcadeForge Deployment Guide

This document provides comprehensive deployment instructions for ArcadeForge on AWS infrastructure.

## Overview

ArcadeForge uses a multi-service architecture deployed on AWS with the following components:

- **Game Editor**: Static React app served via CloudFront CDN
- **API Server**: Node.js application running on ECS Fargate
- **Game Runtime**: Static assets served via CloudFront CDN  
- **Database**: PostgreSQL on RDS with read replicas
- **Cache**: Redis on ElastiCache
- **Asset Storage**: S3 with CloudFront distribution
- **Load Balancer**: Application Load Balancer (ALB)

## Prerequisites

### Required Tools
```bash
# Install required CLI tools
npm install -g aws-cdk@latest
npm install -g @aws-cdk/cli
pip install awscli
brew install terraform  # macOS
brew install kubectl
brew install helm
```

### AWS Configuration
```bash
# Configure AWS credentials
aws configure
# AWS Access Key ID: [Your access key]
# AWS Secret Access Key: [Your secret key]
# Default region name: us-east-1
# Default output format: json

# Verify access
aws sts get-caller-identity
```

### Environment Setup
```bash
# Create environment files
cp .env.example .env.production
cp .env.example .env.staging

# Configure production environment
cat > .env.production << EOF
NODE_ENV=production
DATABASE_URL=postgresql://arcadeforge:${DB_PASSWORD}@${RDS_ENDPOINT}:5432/arcadeforge_prod
REDIS_URL=redis://${ELASTICACHE_ENDPOINT}:6379
AWS_REGION=us-east-1
AWS_S3_BUCKET=arcadeforge-assets-prod
AWS_CLOUDFRONT_DOMAIN=cdn.arcadeforge.com
JWT_SECRET=${JWT_SECRET}
LOG_LEVEL=info
EOF
```

## Infrastructure as Code

### Terraform Configuration

#### VPC and Networking
```hcl
# infrastructure/terraform/vpc.tf
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "arcadeforge-vpc"
    Environment = var.environment
  }
}

resource "aws_subnet" "private" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 1}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "arcadeforge-private-${count.index + 1}"
    Type = "Private"
  }
}

resource "aws_subnet" "public" {
  count                   = 2
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.${count.index + 10}.0/24"
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "arcadeforge-public-${count.index + 1}"
    Type = "Public"
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "arcadeforge-igw"
  }
}

resource "aws_nat_gateway" "main" {
  count         = 2
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id

  tags = {
    Name = "arcadeforge-nat-${count.index + 1}"
  }
}
```

#### RDS Database
```hcl
# infrastructure/terraform/database.tf
resource "aws_db_subnet_group" "main" {
  name       = "arcadeforge-db-subnet-group"
  subnet_ids = aws_subnet.private[*].id

  tags = {
    Name = "ArcadeForge DB subnet group"
  }
}

resource "aws_db_instance" "postgres" {
  identifier = "arcadeforge-postgres"

  engine         = "postgres"
  engine_version = "15.4"
  instance_class = "db.t3.medium"

  allocated_storage     = 100
  max_allocated_storage = 1000
  storage_type          = "gp3"
  storage_encrypted     = true

  db_name  = "arcadeforge"
  username = "arcadeforge"
  password = var.db_password

  vpc_security_group_ids = [aws_security_group.database.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name

  backup_retention_period = 7
  backup_window          = "07:00-09:00"
  maintenance_window     = "sun:09:00-sun:10:00"

  performance_insights_enabled = true
  monitoring_interval         = 60
  monitoring_role_arn         = aws_iam_role.rds_enhanced_monitoring.arn

  deletion_protection = true
  skip_final_snapshot = false

  tags = {
    Name = "ArcadeForge PostgreSQL"
  }
}

resource "aws_db_instance" "postgres_read_replica" {
  identifier = "arcadeforge-postgres-read-replica"

  replicate_source_db = aws_db_instance.postgres.identifier
  instance_class      = "db.t3.medium"

  auto_minor_version_upgrade = true
  publicly_accessible       = false

  tags = {
    Name = "ArcadeForge PostgreSQL Read Replica"
  }
}
```

#### ElastiCache Redis
```hcl
# infrastructure/terraform/cache.tf
resource "aws_elasticache_subnet_group" "main" {
  name       = "arcadeforge-cache-subnet"
  subnet_ids = aws_subnet.private[*].id
}

resource "aws_elasticache_replication_group" "redis" {
  replication_group_id         = "arcadeforge-redis"
  description                  = "Redis cluster for ArcadeForge"
  
  port               = 6379
  parameter_group_name = "default.redis7"
  
  node_type          = "cache.t3.medium"
  num_cache_clusters = 2
  
  subnet_group_name  = aws_elasticache_subnet_group.main.name
  security_group_ids = [aws_security_group.redis.id]
  
  at_rest_encryption_enabled = true
  transit_encryption_enabled = true
  auth_token                = var.redis_auth_token
  
  snapshot_retention_limit = 3
  snapshot_window         = "07:00-09:00"
  
  tags = {
    Name = "ArcadeForge Redis"
  }
}
```

#### S3 and CloudFront
```hcl
# infrastructure/terraform/storage.tf
resource "aws_s3_bucket" "assets" {
  bucket = "arcadeforge-assets-${var.environment}"

  tags = {
    Name        = "ArcadeForge Assets"
    Environment = var.environment
  }
}

resource "aws_s3_bucket_versioning" "assets" {
  bucket = aws_s3_bucket.assets.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "assets" {
  bucket = aws_s3_bucket.assets.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_cloudfront_distribution" "assets" {
  origin {
    domain_name = aws_s3_bucket.assets.bucket_regional_domain_name
    origin_id   = "S3-${aws_s3_bucket.assets.bucket}"

    s3_origin_config {
      origin_access_identity = aws_cloudfront_origin_access_identity.assets.cloudfront_access_identity_path
    }
  }

  enabled             = true
  default_root_object = "index.html"

  default_cache_behavior {
    allowed_methods        = ["DELETE", "GET", "HEAD", "OPTIONS", "PATCH", "POST", "PUT"]
    cached_methods         = ["GET", "HEAD"]
    target_origin_id       = "S3-${aws_s3_bucket.assets.bucket}"
    compress               = true
    viewer_protocol_policy = "redirect-to-https"

    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }

    min_ttl     = 0
    default_ttl = 86400
    max_ttl     = 31536000
  }

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }

  viewer_certificate {
    cloudfront_default_certificate = true
  }

  tags = {
    Name = "ArcadeForge Assets CDN"
  }
}
```

### ECS Configuration

#### ECS Cluster and Service
```hcl
# infrastructure/terraform/ecs.tf
resource "aws_ecs_cluster" "main" {
  name = "arcadeforge"

  setting {
    name  = "containerInsights"
    value = "enabled"
  }

  tags = {
    Name = "ArcadeForge ECS Cluster"
  }
}

resource "aws_ecs_task_definition" "api" {
  family                   = "arcadeforge-api"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = 512
  memory                   = 1024
  execution_role_arn       = aws_iam_role.ecs_execution.arn
  task_role_arn           = aws_iam_role.ecs_task.arn

  container_definitions = jsonencode([
    {
      name  = "api"
      image = "${var.ecr_repository_url}:${var.image_tag}"

      portMappings = [
        {
          containerPort = 8000
          protocol      = "tcp"
        }
      ]

      environment = [
        {
          name  = "NODE_ENV"
          value = "production"
        },
        {
          name  = "PORT"
          value = "8000"
        }
      ]

      secrets = [
        {
          name      = "DATABASE_URL"
          valueFrom = aws_ssm_parameter.database_url.arn
        },
        {
          name      = "REDIS_URL"
          valueFrom = aws_ssm_parameter.redis_url.arn
        },
        {
          name      = "JWT_SECRET"
          valueFrom = aws_ssm_parameter.jwt_secret.arn
        }
      ]

      logConfiguration = {
        logDriver = "awslogs"
        options = {
          awslogs-group         = aws_cloudwatch_log_group.api.name
          awslogs-region        = var.aws_region
          awslogs-stream-prefix = "ecs"
        }
      }

      healthCheck = {
        command = ["CMD-SHELL", "curl -f http://localhost:8000/health || exit 1"]
        interval = 30
        timeout = 5
        retries = 3
      }
    }
  ])
}

resource "aws_ecs_service" "api" {
  name            = "arcadeforge-api"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.api.arn
  desired_count   = 3
  launch_type     = "FARGATE"

  network_configuration {
    security_groups  = [aws_security_group.api.id]
    subnets          = aws_subnet.private[*].id
    assign_public_ip = false
  }

  load_balancer {
    target_group_arn = aws_lb_target_group.api.arn
    container_name   = "api"
    container_port   = 8000
  }

  depends_on = [aws_lb_listener.api]

  lifecycle {
    ignore_changes = [desired_count]
  }
}
```

## Deployment Process

### 1. Infrastructure Deployment

```bash
# Deploy base infrastructure
cd infrastructure/terraform

# Initialize Terraform
terraform init

# Plan deployment
terraform plan -var-file="production.tfvars"

# Apply infrastructure
terraform apply -var-file="production.tfvars"

# Get outputs
terraform output
```

### 2. Database Setup

```bash
# Run database migrations
pnpm db:migrate:prod

# Seed initial data
pnpm db:seed:prod

# Create database backup
aws rds create-db-snapshot \
  --db-instance-identifier arcadeforge-postgres \
  --db-snapshot-identifier arcadeforge-initial-snapshot
```

### 3. Application Deployment

#### Build and Push Docker Images
```bash
# Build production images
docker build -t arcadeforge/api:latest -f apps/api-server/Dockerfile .
docker build -t arcadeforge/game-editor:latest -f apps/game-editor/Dockerfile .

# Tag for ECR
docker tag arcadeforge/api:latest ${ECR_REGISTRY}/arcadeforge-api:${VERSION}
docker tag arcadeforge/game-editor:latest ${ECR_REGISTRY}/arcadeforge-editor:${VERSION}

# Push to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ECR_REGISTRY}
docker push ${ECR_REGISTRY}/arcadeforge-api:${VERSION}
docker push ${ECR_REGISTRY}/arcadeforge-editor:${VERSION}
```

#### Deploy API Service
```bash
# Update ECS service
aws ecs update-service \
  --cluster arcadeforge \
  --service arcadeforge-api \
  --task-definition arcadeforge-api:${TASK_DEFINITION_REVISION} \
  --desired-count 3

# Wait for deployment
aws ecs wait services-stable \
  --cluster arcadeforge \
  --services arcadeforge-api
```

#### Deploy Frontend
```bash
# Build production frontend
cd apps/game-editor
pnpm build

# Sync to S3
aws s3 sync dist/ s3://arcadeforge-frontend-prod/ --delete

# Invalidate CloudFront cache
aws cloudfront create-invalidation \
  --distribution-id ${CLOUDFRONT_DISTRIBUTION_ID} \
  --paths "/*"
```

### 4. Monitoring Setup

#### CloudWatch Dashboards
```bash
# Create custom dashboard
aws cloudwatch put-dashboard \
  --dashboard-name "ArcadeForge-Production" \
  --dashboard-body file://infrastructure/monitoring/dashboard.json
```

#### Alarms Configuration
```bash
# API Error Rate Alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "ArcadeForge-API-ErrorRate" \
  --alarm-description "API error rate exceeds threshold" \
  --metric-name "ErrorRate" \
  --namespace "AWS/ApplicationELB" \
  --statistic "Average" \
  --period 300 \
  --threshold 0.01 \
  --comparison-operator "GreaterThanThreshold" \
  --evaluation-periods 2

# Database Connection Alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "ArcadeForge-DB-Connections" \
  --alarm-description "Database connection count high" \
  --metric-name "DatabaseConnections" \
  --namespace "AWS/RDS" \
  --statistic "Average" \
  --period 300 \
  --threshold 100 \
  --comparison-operator "GreaterThanThreshold" \
  --evaluation-periods 2
```

## Environment Management

### Staging Environment

```bash
# Deploy to staging
export ENVIRONMENT=staging
terraform workspace select staging
terraform apply -var-file="staging.tfvars"

# Deploy application
export ECR_TAG=staging-${COMMIT_SHA}
./scripts/deploy.sh staging
```

### Production Deployment

```bash
# Production deployment with blue-green strategy
export ENVIRONMENT=production
export DEPLOYMENT_STRATEGY=blue-green

# Deploy new version to green environment
./scripts/deploy-blue-green.sh deploy

# Run health checks
./scripts/health-check.sh production

# Switch traffic to green
./scripts/deploy-blue-green.sh switch

# Cleanup old blue environment
./scripts/deploy-blue-green.sh cleanup
```

## Scaling Configuration

### Auto Scaling Policies

```hcl
# ECS Auto Scaling
resource "aws_appautoscaling_target" "api" {
  max_capacity       = 20
  min_capacity       = 3
  resource_id        = "service/arcadeforge/arcadeforge-api"
  scalable_dimension = "ecs:service:DesiredCount"
  service_namespace  = "ecs"
}

resource "aws_appautoscaling_policy" "api_cpu" {
  name               = "api-cpu-scaling"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.api.resource_id
  scalable_dimension = aws_appautoscaling_target.api.scalable_dimension
  service_namespace  = aws_appautoscaling_target.api.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageCPUUtilization"
    }
    target_value = 70.0
  }
}
```

### Database Scaling

```bash
# Scale RDS instance
aws rds modify-db-instance \
  --db-instance-identifier arcadeforge-postgres \
  --db-instance-class db.r5.xlarge \
  --apply-immediately

# Add read replica in different region
aws rds create-db-instance-read-replica \
  --db-instance-identifier arcadeforge-postgres-read-replica-west \
  --source-db-instance-identifier arcadeforge-postgres \
  --db-instance-class db.t3.medium
```

## Backup and Recovery

### Database Backups
```bash
# Automated backups are configured in Terraform
# Manual snapshot
aws rds create-db-snapshot \
  --db-instance-identifier arcadeforge-postgres \
  --db-snapshot-identifier manual-snapshot-$(date +%Y%m%d%H%M%S)

# Restore from snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier arcadeforge-postgres-restored \
  --db-snapshot-identifier manual-snapshot-20240101120000
```

### S3 Asset Backups
```bash
# Cross-region replication is configured in Terraform
# Manual backup
aws s3 sync s3://arcadeforge-assets-prod/ s3://arcadeforge-assets-backup/ --delete
```

## Security Considerations

### SSL/TLS Configuration
- All traffic encrypted in transit (TLS 1.3)
- RDS encryption at rest enabled
- S3 server-side encryption enabled
- ElastiCache transit encryption enabled

### Network Security
- VPC with private subnets for database and cache
- Security groups with least privilege access
- WAF rules for API protection
- VPC Flow Logs enabled

### Access Control
- IAM roles with minimal required permissions
- Secrets stored in AWS Systems Manager Parameter Store
- ECR repository with image vulnerability scanning
- CloudTrail logging for audit trails

## Troubleshooting

### Common Issues

#### ECS Service Won't Start
```bash
# Check service events
aws ecs describe-services --cluster arcadeforge --services arcadeforge-api

# Check task definition
aws ecs describe-task-definition --task-definition arcadeforge-api

# Check logs
aws logs get-log-events \
  --log-group-name /ecs/arcadeforge-api \
  --log-stream-name ecs/api/$(aws ecs list-tasks --cluster arcadeforge --service-name arcadeforge-api --query 'taskArns[0]' --output text | cut -d'/' -f3)
```

#### Database Connection Issues
```bash
# Check RDS status
aws rds describe-db-instances --db-instance-identifier arcadeforge-postgres

# Test connectivity from ECS task
aws ecs run-task \
  --cluster arcadeforge \
  --task-definition arcadeforge-debug \
  --overrides '{"containerOverrides":[{"name":"debug","command":["pg_isready","-h","${RDS_ENDPOINT}","-p","5432"]}]}'
```

#### Performance Issues
```bash
# Check CloudWatch metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/ECS \
  --metric-name CPUUtilization \
  --dimensions Name=ServiceName,Value=arcadeforge-api Name=ClusterName,Value=arcadeforge \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-01T23:59:59Z \
  --period 300 \
  --statistics Average

# Check database performance
aws rds describe-db-instances --db-instance-identifier arcadeforge-postgres \
  --query 'DBInstances[0].{CPUUtilization:ProcessorFeatures,Connections:DbInstanceStatus}'
```

This deployment guide provides a comprehensive foundation for running ArcadeForge in production with proper monitoring, scaling, and security configurations.
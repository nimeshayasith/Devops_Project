# High Availability Web Application on AWS

A production-ready, highly available web application deployed on AWS using CloudFormation, featuring auto-scaling, load balancing, and comprehensive monitoring.

![AWS](https://img.shields.io/badge/AWS-CloudFormation-orange)
![Status](https://img.shields.io/badge/Status-Production%20Ready-success)
![License](https://img.shields.io/badge/License-MIT-blue)

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Deployment Guide](#deployment-guide)
- [Configuration](#configuration)
- [Testing](#testing)
- [Monitoring](#monitoring)
- [What I Learned](#what-i-learned)
- [Troubleshooting](#troubleshooting)
- [Cost Estimation](#cost-estimation)
- [Clean Up](#clean-up)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Overview

This project demonstrates a scalable, fault-tolerant web application infrastructure on AWS. The application automatically scales based on demand and maintains high availability across multiple Availability Zones.

**Live Demo:** Visit your ALB DNS to see the application in action!

## 🏗️ Architecture

```
                          Internet
                             |
                    [Application Load Balancer]
                             |
                    [Target Group]
                      /          \
        [Auto Scaling Group]
           /        |        \
    [EC2 Instance] [EC2 Instance] [EC2 Instance]
      (AZ-1a)        (AZ-1b)        (AZ-1a)
                             |
                    [CloudWatch Monitoring]
                             |
                    [SNS Notifications]
```

### Components:

- **Application Load Balancer (ALB)**: Distributes incoming traffic across multiple EC2 instances
- **Auto Scaling Group**: Automatically adjusts instance count based on demand (1-4 instances)
- **Launch Template**: Defines instance configuration with Amazon Linux 2023
- **Target Group**: Health checks and routes traffic to healthy instances
- **CloudWatch**: Monitors metrics and triggers alarms
- **SNS**: Sends email notifications for scaling events and alarms
- **IAM Roles**: Enables SSM Session Manager and CloudWatch agent
- **Security Groups**: Controls inbound/outbound traffic

## ✨ Features

### High Availability
- ✅ Multi-AZ deployment for fault tolerance
- ✅ Automatic instance replacement on failure
- ✅ Health checks with automatic traffic routing
- ✅ Zero-downtime rolling updates

### Auto Scaling
- ✅ CPU-based scaling (target: 50%)
- ✅ Request count-based scaling (target: 1000 req/instance)
- ✅ Configurable min/max/desired capacity
- ✅ Scale-in and scale-out policies

### Security
- ✅ IMDSv2 enforced for metadata security
- ✅ Security groups with least privilege access
- ✅ SSM Session Manager (no SSH keys required)
- ✅ IAM roles with CloudWatch and SSM permissions

### Monitoring & Alerts
- ✅ High CPU utilization alarm (>80%)
- ✅ Unhealthy host detection
- ✅ CloudWatch metrics collection
- ✅ Email notifications via SNS

### Web Application
- ✅ Beautiful, responsive UI with gradients
- ✅ Displays instance ID, AZ, and region
- ✅ Health check endpoint
- ✅ Apache web server with custom page

## 📦 Prerequisites

Before deploying this application, ensure you have:

- **AWS Account** with appropriate permissions
- **AWS CLI** installed and configured
- **EC2 Key Pair** (optional - SSM available)
- **VPC** with at least 2 public subnets in different AZs
- **Email address** for notifications

### Required IAM Permissions:
- CloudFormation: Full access
- EC2: Create/manage instances, security groups, launch templates
- ELB: Create/manage load balancers and target groups
- Auto Scaling: Create/manage auto scaling groups
- IAM: Create/manage roles and instance profiles
- SNS: Create/manage topics and subscriptions
- CloudWatch: Create/manage alarms

## 🚀 Deployment Guide

### Method 1: AWS Console

1. **Navigate to CloudFormation**
   ```
   AWS Console → CloudFormation → Create Stack
   ```

2. **Upload Template**
   - Select "Upload a template file"
   - Choose `enhanced-ha-app.yaml`
   - Click "Next"

3. **Specify Parameters**
   ```
   Stack Name: my-ha-web-app
   
   Network Configuration:
   - VPC: Select your VPC
   - Subnets: Select 2+ public subnets (different AZs)
   - SSHLocation: Your IP/32 (recommended)
   
   EC2 Configuration:
   - InstanceType: t3.micro
   - KeyName: Your EC2 key pair
   
   Auto Scaling:
   - DesiredCapacity: 2
   - MinSize: 1
   - MaxSize: 4
   - CPUTargetValue: 50
   
   Notifications:
   - OperatorEmail: your-email@example.com
   
   Environment: Production
   ```

4. **Configure Stack Options**
   - Add tags (optional)
   - Enable termination protection (recommended)
   - Click "Next"

5. **Review and Create**
   - Check "I acknowledge that AWS CloudFormation might create IAM resources"
   - Click "Create Stack"

6. **Wait for Completion** (5-10 minutes)
   - Monitor the Events tab
   - Status will change to `CREATE_COMPLETE`

7. **Confirm SNS Subscription**
   - Check your email for confirmation link
   - Click to confirm notifications

### Method 2: AWS CLI

```bash
aws cloudformation create-stack \
  --stack-name my-ha-web-app \
  --template-body file://enhanced-ha-app.yaml \
  --parameters \
    ParameterKey=VPC,ParameterValue=vpc-xxxxx \
    ParameterKey=Subnets,ParameterValue="subnet-xxxxx\,subnet-yyyyy" \
    ParameterKey=KeyName,ParameterValue=your-key-pair \
    ParameterKey=InstanceType,ParameterValue=t3.micro \
    ParameterKey=OperatorEmail,ParameterValue=your-email@example.com \
    ParameterKey=SSHLocation,ParameterValue=0.0.0.0/0 \
    ParameterKey=DesiredCapacity,ParameterValue=2 \
    ParameterKey=MinSize,ParameterValue=1 \
    ParameterKey=MaxSize,ParameterValue=4 \
    ParameterKey=CPUTargetValue,ParameterValue=50 \
    ParameterKey=Environment,ParameterValue=Production \
  --capabilities CAPABILITY_NAMED_IAM \
  --tags Key=Project,Value=HA-WebApp
```

### Verify Deployment

```bash
# Check stack status
aws cloudformation describe-stacks \
  --stack-name my-ha-web-app \
  --query 'Stacks[0].StackStatus'

# Get ALB URL
aws cloudformation describe-stacks \
  --stack-name my-ha-web-app \
  --query 'Stacks[0].Outputs[?OutputKey==`ApplicationURL`].OutputValue' \
  --output text
```

## ⚙️ Configuration

### Stack Parameters

| Parameter | Description | Default | Allowed Values |
|-----------|-------------|---------|----------------|
| `InstanceType` | EC2 instance type | t3.micro | t3.micro, t3.small, t3.medium, t3.large |
| `DesiredCapacity` | Initial instance count | 2 | 1-10 |
| `MinSize` | Minimum instances | 1 | 1-10 |
| `MaxSize` | Maximum instances | 4 | 1-10 |
| `CPUTargetValue` | Target CPU % for scaling | 50 | 30-90 |
| `SSHLocation` | CIDR for SSH access | 0.0.0.0/0 | Valid CIDR |
| `Environment` | Environment tag | Production | Development, Staging, Production |

### Updating the Stack

To modify parameters:

```bash
aws cloudformation update-stack \
  --stack-name my-ha-web-app \
  --template-body file://enhanced-ha-app.yaml \
  --parameters ParameterKey=DesiredCapacity,ParameterValue=3 \
  --capabilities CAPABILITY_NAMED_IAM
```

## 🧪 Testing

### 1. Access the Application

```bash
# Get the URL
ALB_DNS=$(aws cloudformation describe-stacks \
  --stack-name my-ha-web-app \
  --query 'Stacks[0].Outputs[?OutputKey==`ApplicationURL`].OutputValue' \
  --output text)

# Test HTTP response
curl $ALB_DNS
```

Open the URL in your browser to see the beautiful web interface!

### 2. Test Load Balancing

Refresh the page multiple times and observe different instance IDs appearing - this confirms load balancing is working.

```bash
# Make multiple requests
for i in {1..10}; do
  curl -s $ALB_DNS | grep "Instance ID"
  sleep 1
done
```

### 3. Test High Availability

Terminate an instance and watch Auto Scaling automatically launch a replacement:

```bash
# Get instance ID
INSTANCE_ID=$(aws ec2 describe-instances \
  --filters "Name=tag:aws:cloudformation:stack-name,Values=my-ha-web-app" \
            "Name=instance-state-name,Values=running" \
  --query 'Reservations[0].Instances[0].InstanceId' \
  --output text)

# Terminate instance
aws ec2 terminate-instances --instance-ids $INSTANCE_ID

# Watch Auto Scaling replace it
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names my-ha-web-app-ASG \
  --query 'AutoScalingGroups[0].Instances[*].[InstanceId,LifecycleState]'
```

You should receive an email notification about the termination and launch.

### 4. Test Auto Scaling

Generate load to trigger scaling:

```bash
# Install stress tool (on EC2 instance)
sudo yum install -y stress

# Generate CPU load
stress --cpu 4 --timeout 300s

# Watch scaling activity
aws autoscaling describe-scaling-activities \
  --auto-scaling-group-name my-ha-web-app-ASG \
  --max-records 5
```

## 📊 Monitoring

### CloudWatch Dashboards

1. Go to **CloudWatch → Dashboards**
2. Create a custom dashboard with:
   - ALB request count
   - Target response time
   - EC2 CPU utilization
   - Healthy/unhealthy host count

### View Metrics

```bash
# CPU utilization
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=AutoScalingGroupName,Value=my-ha-web-app-ASG \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Average

# ALB request count
aws cloudwatch get-metric-statistics \
  --namespace AWS/ApplicationELB \
  --metric-name RequestCount \
  --dimensions Name=LoadBalancer,Value=<your-alb-full-name> \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Sum
```

### Active Alarms

- **HighCPUAlarm**: Triggers when average CPU > 80% for 10 minutes
- **UnhealthyHostAlarm**: Triggers when target group has unhealthy hosts

## 💡 What I Learned

### Technical Skills Gained

#### 1. **CloudFormation Infrastructure as Code**
- Learned to write declarative infrastructure code using YAML
- Understood CloudFormation intrinsic functions (`!Ref`, `!GetAtt`, `!Sub`, `!Join`)
- Mastered parameter handling and stack outputs
- Implemented cross-stack references with exports
- Handled resource dependencies and update policies

**Key Takeaway**: Infrastructure as Code enables version control, repeatability, and faster deployments compared to manual console work.

#### 2. **AWS Auto Scaling Deep Dive**
- Configured Auto Scaling Groups with multiple scaling policies
- Implemented target tracking scaling (CPU and request-based)
- Understood health check grace periods and rolling updates
- Learned about scale-in protection and cooldown periods

**Challenge Overcome**: Initially struggled with instances being terminated too quickly during updates. Solved by implementing `UpdatePolicy` with `MinInstancesInService` to ensure zero downtime.

#### 3. **Load Balancer Architecture**
- Designed Application Load Balancer for Layer 7 routing
- Configured target groups with health checks
- Implemented session stickiness for stateful applications
- Understood deregistration delay for graceful shutdowns

**Insight**: Health checks are critical - proper configuration (interval, timeout, thresholds) prevents false positives and ensures only healthy instances receive traffic.

#### 4. **AWS Instance Metadata Service (IMDS)**
- Learned about the special IP `169.254.169.254`
- Implemented IMDSv2 for enhanced security (session-oriented)
- Used metadata to create dynamic, instance-aware web pages
- Understood metadata security implications

**Security Learning**: IMDSv2 with `HttpTokens: required` prevents SSRF attacks by requiring a PUT request before accessing metadata.

#### 5. **IAM Roles and Security**
- Created IAM roles with least privilege principle
- Attached managed policies (SSM, CloudWatch)
- Implemented instance profiles for EC2
- Understood trust relationships and assume role policies

**Best Practice**: Always use IAM roles for EC2 instead of embedding credentials. SSM Session Manager eliminates the need for SSH keys entirely.

#### 6. **CloudWatch Monitoring and Alarms**
- Set up custom metrics and alarms
- Configured SNS for multi-channel notifications
- Learned metric namespaces and dimensions
- Implemented composite alarms for complex scenarios

**Pro Tip**: Use multiple data points for alarms (`EvaluationPeriods: 2`) to avoid false alarms from temporary spikes.

#### 7. **VPC Networking Fundamentals**
- Understood public vs. private subnets
- Configured security groups with proper ingress/egress rules
- Learned about multi-AZ deployment for high availability
- Implemented security group chaining (ALB → EC2)

**Design Decision**: Deployed instances in public subnets for simplicity, but production should use private subnets with NAT Gateway.

#### 8. **User Data and Bootstrap Scripts**
- Wrote bash scripts for automated instance configuration
- Installed and configured Apache web server
- Implemented CloudWatch agent for custom metrics
- Used `set -e` for fail-fast script execution

**Lesson Learned**: User data only runs once at launch. For updates, use Systems Manager Run Command or update the Launch Template.

#### 9. **High Availability Patterns**
- Implemented multi-AZ deployment (spread across zones)
- Configured health checks at multiple layers (ELB + ASG)
- Designed for automatic failure recovery
- Used load balancer for traffic distribution

**Architecture Principle**: High availability isn't just about redundancy - it's about detection, isolation, and automatic recovery.

#### 10. **Cost Optimization Strategies**
- Chose t3.micro instances for cost-effective testing
- Implemented auto scaling to match demand
- Used target tracking to avoid over-provisioning
- Understood pricing for ALB, data transfer, and EC2

**Budget Insight**: ~$35/month for basic setup. Can reduce by using spot instances or scheduling off-hours shutdown for dev environments.

### DevOps Best Practices

- **Version Control**: All infrastructure code in Git
- **Parameterization**: Made stack reusable across environments
- **Tagging Strategy**: Consistent tags for cost tracking and organization
- **Documentation**: Comprehensive README and inline comments
- **Testing**: Automated testing of HA and scaling capabilities

### Challenges Faced and Solutions

| Challenge | Solution |
|-----------|----------|
| Instances terminating during updates | Implemented `UpdatePolicy` with rolling updates |
| Hard to debug failed deployments | Added comprehensive outputs and CloudWatch logs |
| Security group rules too permissive | Changed SSH from 0.0.0.0/0 to specific IP |
| Manual SNS subscription tedious | Automated in template (still requires email confirmation) |
| CloudWatch agent not reporting metrics | Added proper IAM policies and configuration file |

### Future Enhancements

- [ ] Add HTTPS/SSL with ACM certificate
- [ ] Implement blue-green deployment strategy
- [ ] Add RDS database backend
- [ ] Use private subnets with NAT Gateway
- [ ] Implement AWS Backup for disaster recovery
- [ ] Add AWS WAF for web application firewall
- [ ] Set up CodePipeline for CI/CD
- [ ] Implement ElastiCache for session management
- [ ] Add Route 53 custom domain
- [ ] Containerize with ECS/Fargate

### Resources That Helped Me

- [AWS CloudFormation Documentation](https://docs.aws.amazon.com/cloudformation/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Auto Scaling Best Practices](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-best-practices.html)
- [AWS Solutions Library](https://aws.amazon.com/solutions/)

## 🐛 Troubleshooting

### Stack Creation Fails

**Problem**: Stack stuck in `CREATE_IN_PROGRESS` or `ROLLBACK_IN_PROGRESS`

**Solutions**:
1. Check Events tab for detailed error messages
2. Verify you have 2+ subnets in different AZs
3. Ensure EC2 key pair exists in the same region
4. Verify email address is valid
5. Check IAM permissions

### Instances Not Healthy

**Problem**: Target group shows unhealthy instances

**Solutions**:
```bash
# Check Apache status
sudo systemctl status httpd

# Restart Apache
sudo systemctl restart httpd

# Check security group rules
aws ec2 describe-security-groups --group-ids sg-xxxxx

# View target health
aws elbv2 describe-target-health --target-group-arn arn:aws:...
```

### Can't Access Application

**Problem**: ALB URL returns timeout or error

**Solutions**:
1. Verify instances are in public subnets with internet gateway
2. Check security group allows port 80 from 0.0.0.0/0
3. Ensure instances are healthy in target group
4. Wait for health check grace period (5 minutes)

### Auto Scaling Not Working

**Problem**: Instances not scaling up/down

**Solutions**:
```bash
# Check scaling activities
aws autoscaling describe-scaling-activities \
  --auto-scaling-group-name my-ha-web-app-ASG

# View scaling policies
aws autoscaling describe-policies \
  --auto-scaling-group-name my-ha-web-app-ASG

# Check CloudWatch alarms
aws cloudwatch describe-alarms
```

### Not Receiving Email Notifications

**Problem**: No SNS notifications arriving

**Solutions**:
1. Check spam/junk folder
2. Confirm SNS subscription via email link
3. Verify email address in stack parameters
4. Check SNS topic subscriptions in console

## 💰 Cost Estimation

Monthly costs (us-east-1 region):

| Service | Usage | Monthly Cost |
|---------|-------|--------------|
| EC2 (t3.micro × 2) | 730 hours × 2 | ~$15.00 |
| Application Load Balancer | 730 hours | ~$16.43 |
| Data Transfer | 10 GB out | ~$0.90 |
| CloudWatch | Basic monitoring | ~$0.00 |
| **Total** | | **~$32-35** |

**Cost Optimization Tips**:
- Use t3a instances (10% cheaper)
- Implement scheduled scaling (shut down at night)
- Use Savings Plans or Reserved Instances
- Monitor with AWS Cost Explorer

## 🧹 Clean Up

To avoid ongoing charges, delete the stack:

### Via Console:
1. Go to CloudFormation
2. Select your stack
3. Click "Delete"
4. Confirm deletion

### Via CLI:
```bash
aws cloudformation delete-stack --stack-name my-ha-web-app

# Wait for deletion to complete
aws cloudformation wait stack-delete-complete --stack-name my-ha-web-app
```

**Note**: This will delete all resources created by the stack (EC2 instances, ALB, security groups, etc.)

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- AWS Documentation and best practices guides
- AWS Community Builders for insights
- CloudFormation sample templates
- Open source community

## 📞 Contact

- **Author**: [Your Name]
- **Email**: nimeshayasith@gmail.com
- **LinkedIn**: [Your LinkedIn Profile]
- **GitHub**: [Your GitHub Profile]

---

**⭐ If you found this project helpful, please give it a star!**

**📚 For more AWS projects, check out my other repositories!**

---

*Last Updated: October 2025*

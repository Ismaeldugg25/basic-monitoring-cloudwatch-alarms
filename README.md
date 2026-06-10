# Basic AWS Resources Monitoring with CloudWatch Alarms

Basic monitoring system using Amazon CloudWatch Alarms and SNS for automated infrastructure notifications across EC2, RDS, and ALB.

---

## Problem

Organizations need visibility into their AWS resources to identify operational issues, but without proper monitoring, performance problems can go unnoticed until they cause service disruptions. Manual monitoring is time-consuming and unreliable, especially outside business hours. Small teams particularly need automated monitoring solutions that alert them when metrics exceed predefined thresholds, helping maintain service reliability while minimising operational overhead.

---

## Solution

This project implements basic monitoring using Amazon CloudWatch and SNS to automatically track key metrics and send email notifications when thresholds are breached. CloudWatch continuously collects metrics from AWS resources, CloudWatch Alarms evaluate those metrics against defined thresholds, and SNS delivers notifications via email. The entire infrastructure is deployed using **Terraform**, following an Infrastructure as Code approach.

---

## Architecture

Insert image here 

---

## Tools & Services Used

- Amazon CloudWatch
- Amazon SNS (Simple Notification Service)
- Amazon EC2 (monitored resource)
- Amazon RDS (monitored resource)
- Amazon ALB (monitored resource)
- CloudWatch Dashboard
- Terraform
- AWS CLI

---

## Prerequisites

- AWS account with CloudWatch and SNS permissions
- AWS CLI v2 installed and configured
- Terraform >= 1.6.0 installed
- An email address for receiving notifications

---

## Terraform Deployment 
This project was deployed entirely using Terraform. All AWS resources were defined as code, planned, and applied without manually clicking through the AWS Console.

Everything was written in Terraform across 5 files:
├── versions.tf        ← Terraform + provider version requirements
├── main.tf            ← All AWS resources
├── variables.tf       ← Input variable definitions with validation
├── outputs.tf         ← Values and CLI commands printed after deployment
└── terraform.tfvars   ← Your personal values (excluded from GitHub)

## Steps

### 1. Create SNS Topic for Alert Notifications

Amazon SNS (Simple Notification Service) provides a reliable, scalable messaging service that delivers notifications to multiple endpoints simultaneously. Creating an SNS topic establishes the communication channel that CloudWatch Alarms will use to send alerts when metric thresholds are breached.
IMAGE

### 2. Subscribe Email Address to SNS Topic

Email subscriptions provide immediate notification delivery to operations teams, ensuring critical alerts reach the right people quickly. You must confirm this subscription before alarms can deliver notifications, ensuring only authorized recipients receive alerts.

IMAGE 

### 3. Create CloudWatch Alarm for High CPU Usage

CloudWatch Alarms continuously evaluate metrics against defined thresholds, providing automated monitoring without manual intervention.

IMAGE

This alarm monitors average CPU utilization across all EC2 instances in your account, triggering when usage exceeds 80% for two consecutive 5-minute periods. The configuration balances sensitivity with stability, reducing false alarms while ensuring genuine performance issues are detected promptly.

### 4. Create CloudWatch Alarm for Application Load Balancer Response Time

Application performance monitoring through response time metrics helps identify user experience degradation before it impacts business operations. Load balancer latency reflects the health of your entire application stack, including network performance, backend processing time, and resource availability.

IMAGE 

This alarm detects when application response times exceed 1 second for three consecutive periods, indicating potential performance degradation that could affect user satisfaction and business metrics.

### 5. Create CloudWatch Alarm for RDS Database Connections

Database connection monitoring prevents connection pool exhaustion and identifies application scaling requirements. High connection counts often indicate inefficient connection management or unexpected traffic spikes that require immediate attention to maintain application availability.

IMAGE

This alarm monitors database connection utilization across all RDS instances, helping prevent connection exhaustion scenarios that could cause application failures or performance degradation.

---

## Validation & Testing

### 1. Verify SNS Topic and Subscription

List SNS topics to confirm creation:

```bash
aws sns list-topics \
  --query "Topics[?contains(TopicArn, '$SNS_TOPIC_NAME')]"
```
IMAGE 
Check subscription status:

```bash
aws sns list-subscriptions-by-topic \
  --topic-arn <your-sns-topic-arn> \
  --query "Subscriptions[0].{Status:SubscriptionArn,Protocol:Protocol,Endpoint:Endpoint}"
```
IMAGE

### 2. Verify CloudWatch Alarms

List all created alarms:

```bash
aws cloudwatch describe-alarms \
  --alarm-name-prefix "High" \
  --query "MetricAlarms[].{Name:AlarmName,State:StateValue,Threshold:Threshold}"
```
IMAGE
Expected output: All three alarms should appear with `INSUFFICIENT_DATA` or `OK` state and correct threshold values.

### 3. Test Alarm Notification

Manually trigger the alarm to test the full notification pipeline:

```bash
aws cloudwatch set-alarm-state \
  --alarm-name "HighCPUUtilization-<your-suffix>" \
  --state-value ALARM \
  --state-reason "Testing alarm notification system" \
  --region us-west-2
```
IMAGE 

## Steps

### 1. Create SNS Topic for Alert Notifications

Amazon SNS (Simple Notification Service) provides a reliable, scalable messaging service that delivers notifications to multiple endpoints simultaneously. Creating an SNS topic establishes the communication channel that CloudWatch Alarms will use to send alerts when metric thresholds are breached.

### 2. Subscribe Email Address to SNS Topic

Email subscriptions provide immediate notification delivery to operations teams, ensuring critical alerts reach the right people quickly. You must confirm this subscription before alarms can deliver notifications, ensuring only authorized recipients receive alerts.

### 3. Create CloudWatch Alarm for High CPU Usage

CloudWatch Alarms continuously evaluate metrics against defined thresholds, providing automated monitoring without manual intervention.

This alarm monitors average CPU utilization across all EC2 instances in your account, triggering when usage exceeds 80% for two consecutive 5-minute periods. The configuration balances sensitivity with stability, reducing false alarms while ensuring genuine performance issues are detected promptly.

### 4. Create CloudWatch Alarm for Application Load Balancer Response Time

Application performance monitoring through response time metrics helps identify user experience degradation before it impacts business operations. Load balancer latency reflects the health of your entire application stack, including network performance, backend processing time, and resource availability.

This alarm detects when application response times exceed 1 second for three consecutive periods, indicating potential performance degradation that could affect user satisfaction and business metrics.

### 5. Create CloudWatch Alarm for RDS Database Connections

Database connection monitoring prevents connection pool exhaustion and identifies application scaling requirements. High connection counts often indicate inefficient connection management or unexpected traffic spikes that require immediate attention to maintain application availability.

This alarm monitors database connection utilization across all RDS instances, helping prevent connection exhaustion scenarios that could cause application failures or performance degradation.

---

## Validation & Testing

### 1. Verify SNS Topic and Subscription

List SNS topics to confirm creation:

```bash
aws sns list-topics \
  --query "Topics[?contains(TopicArn, '$SNS_TOPIC_NAME')]"
```

Check subscription status:

```bash
aws sns list-subscriptions-by-topic \
  --topic-arn <your-sns-topic-arn> \
  --query "Subscriptions[0].{Status:SubscriptionArn,Protocol:Protocol,Endpoint:Endpoint}"
```

### 2. Verify CloudWatch Alarms

List all created alarms:

```bash
aws cloudwatch describe-alarms \
  --alarm-name-prefix "High" \
  --query "MetricAlarms[].{Name:AlarmName,State:StateValue,Threshold:Threshold}"
```

Expected output: All three alarms should appear with `INSUFFICIENT_DATA` or `OK` state and correct threshold values.

### 3. Test Alarm Notification

Manually trigger the alarm to test the full notification pipeline:

```bash
aws cloudwatch set-alarm-state \
  --alarm-name "HighCPUUtilization-<your-suffix>" \
  --state-value ALARM \
  --state-reason "Testing alarm notification system" \
  --region us-west-2
```
EmaiL received within 30 seconds of triggering the alarm.

IMAGE 
---

## Links

- GitHub: [basic-monitoring-cloudwatch-alarms](https://github.com/Ismaeldugg25/basic-monitoring-cloudwatch-alarms)

---

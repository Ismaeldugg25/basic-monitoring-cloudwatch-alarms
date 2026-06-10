# AWS Resources Monitoring with CloudWatch Alarms
<img width="800" height="400" alt="Nexus-feat-img2" src="https://github.com/user-attachments/assets/88bf90a3-b051-4844-bc5e-a258d1258749" />



Basic monitoring system using Amazon CloudWatch Alarms and SNS for automated infrastructure notifications across EC2, RDS, and ALB.

## Problem

Organizations need visibility into their AWS resources to identify operational issues, but without proper monitoring, performance problems can go unnoticed until they cause service disruptions. Manual monitoring is time-consuming and unreliable, especially outside business hours. Small teams particularly need automated monitoring solutions that alert them when metrics exceed predefined thresholds, helping maintain service reliability while minimising operational overhead.

## Solution

This project implements basic monitoring using Amazon CloudWatch and SNS to automatically track key metrics and send email notifications when thresholds are breached. CloudWatch continuously collects metrics from AWS resources, CloudWatch Alarms evaluate those metrics against defined thresholds, and SNS delivers notifications via email. The entire infrastructure is deployed using **Terraform**, following an Infrastructure as Code approach.

## Architecture

<img width="662" height="659" alt="image-10" src="https://github.com/user-attachments/assets/d910d7ce-9a7e-485b-9ddb-e12da1a72586" />


## Tools & Services Used

- Amazon CloudWatch
- Amazon SNS (Simple Notification Service)
- Amazon EC2 (monitored resource)
- Amazon RDS (monitored resource)
- Amazon ALB (monitored resource)
- CloudWatch Dashboard
- Terraform
- AWS CLI

### Deployed Environment

Three CloudWatch alarms watching three different layers of a typical web application:

- **EC2 CPU alarm** — fires if any server is working too hard (above 80% CPU for 10 minutes)
- **ALB Response Time alarm** — fires if the app starts responding slowly to users (above 1 second for 15 minutes)
- **RDS Connections alarm** — fires if the database is getting overwhelmed (above 80 connections for 10 minutes)

All three alarms send notifications through a single **SNS Topic**, which delivers emails to your inbox. A **CloudWatch Dashboard** was also created to visualise all three metrics in one place.

## Terraform Deployment 
This project was deployed entirely using Terraform. All AWS resources were defined as code, planned, and applied without manually clicking through the AWS Console.

Everything was written in Terraform across 5 files:
- `versions.tf` — provider versions and default tags
- `variables.tf` — all configurable settings with validation rules
- `main.tf` — the actual AWS resources
- `outputs.tf` — useful info and CLI commands printed after deployment
- `terraform.tfvars` —  personal values like email address

## Steps

### 1. Create SNS Topic for Alert Notifications

Amazon SNS (Simple Notification Service) provides a reliable, scalable messaging service that delivers notifications to multiple endpoints simultaneously. Creating an SNS topic establishes the communication channel that CloudWatch Alarms will use to send alerts when metric thresholds are breached.

<img width="754" height="562" alt="image-11" src="https://github.com/user-attachments/assets/ccacdd22-a643-4de3-8ba1-c5a73f74b68a" />


### 2. Subscribe Email Address to SNS Topic

Email subscriptions provide immediate notification delivery to operations teams, ensuring critical alerts reach the right people quickly. You must confirm this subscription before alarms can deliver notifications, ensuring only authorized recipients receive alerts.

<img width="776" height="254" alt="image-12" src="https://github.com/user-attachments/assets/748167ed-2687-4954-929c-beadba82adfe" />


### 3. Create CloudWatch Alarm for High CPU Usage

CloudWatch Alarms continuously evaluate metrics against defined thresholds, providing automated monitoring without manual intervention.

<img width="816" height="549" alt="image-13" src="https://github.com/user-attachments/assets/275962e7-9dac-4387-a381-5061b21c7d8b" />


This alarm monitors average CPU utilization across all EC2 instances in your account, triggering when usage exceeds 80% for two consecutive 5-minute periods. The configuration balances sensitivity with stability, reducing false alarms while ensuring genuine performance issues are detected promptly.

### 4. Create CloudWatch Alarm for Application Load Balancer Response Time

Application performance monitoring through response time metrics helps identify user experience degradation before it impacts business operations. Load balancer latency reflects the health of your entire application stack, including network performance, backend processing time, and resource availability.

<img width="742" height="502" alt="image-14" src="https://github.com/user-attachments/assets/b0075cfa-a2a2-4e30-9342-20ab62660b23" />


This alarm detects when application response times exceed 1 second for three consecutive periods, indicating potential performance degradation that could affect user satisfaction and business metrics.

### 5. Create CloudWatch Alarm for RDS Database Connections

Database connection monitoring prevents connection pool exhaustion and identifies application scaling requirements. High connection counts often indicate inefficient connection management or unexpected traffic spikes that require immediate attention to maintain application availability.

<img width="762" height="497" alt="image-15" src="https://github.com/user-attachments/assets/7e7bf356-a8d4-41bd-80db-32c76396c659" />


This alarm monitors database connection utilization across all RDS instances, helping prevent connection exhaustion scenarios that could cause application failures or performance degradation.

## Validation & Testing

### 1. Verify SNS Topic and Subscription

List SNS topics to confirm creation:

```bash
aws sns list-topics \
  --query "Topics[?contains(TopicArn, '$SNS_TOPIC_NAME')]"
```
<img width="597" height="112" alt="image-16" src="https://github.com/user-attachments/assets/f945b81c-9018-4e0f-acc6-97158b9d2c04" />

 
Check subscription status:

```bash
aws sns list-subscriptions-by-topic \
  --topic-arn <your-sns-topic-arn> \
  --query "Subscriptions[0].{Status:SubscriptionArn,Protocol:Protocol,Endpoint:Endpoint}"
```
<img width="616" height="270" alt="image-17" src="https://github.com/user-attachments/assets/06fc4b9a-b684-42e0-98ef-cea41a8b5ce5" />


### 2. Verify CloudWatch Alarms

List all created alarms:

```bash
aws cloudwatch describe-alarms \
  --alarm-name-prefix "High" \
  --query "MetricAlarms[].{Name:AlarmName,State:StateValue,Threshold:Threshold}"
```
<img width="732" height="283" alt="image-18" src="https://github.com/user-attachments/assets/286635b1-11ab-4cdc-aab1-6c79fab57048" />


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
<img width="1492" height="619" alt="image-19" src="https://github.com/user-attachments/assets/b6aa727c-a0eb-4b10-8ff8-72e8b0366956" />



## Links

- GitHub: [basic-monitoring-cloudwatch-alarms](https://github.com/Ismaeldugg25/basic-monitoring-cloudwatch-alarms)

---

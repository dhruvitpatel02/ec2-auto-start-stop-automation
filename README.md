# ec2-auto-start-stop-automation

# Auto-Pause and Resume EC2 Instances Based on Office Hours (AWS Native)

This project automates EC2 instance start/stop based on custom hours using only AWS-native services — no external scripts or tools. It also sends real-time notifications using Amazon SNS.

## 🧭 Workflow Overview

```text
EventBridge ──> Lambda ──> EC2
                     ↓
                    SNS ──> Email

                 
- AWS Lambda
- Amazon EC2
- Amazon EventBridge (CloudWatch Events)
- Amazon SNS
- IAM Roles & Policies
- Amazon CloudWatch Logs

- Automatically starts EC2 instances at 11:05 PM IST every day
- Automatically stops EC2 instances at 11:10 PM IST every day
- Sends email notifications when instances start/stop
- Tag-based targeting for flexibility and scalability

🔹 Setup Instructions (Step-by-Step)
- Tag instance
- Create IAM role
- Create Lambda functions (code inline)
- Create EventBridge rules
- Create SNS topic and confirm email subscription

# /lambda/start_instance.py
import boto3

ec2 = boto3.resource('ec2')
sns = boto3.client('sns')
sns_topic_arn = 'arn:aws:sns:us-east-1:476114139407:ec2-scheduler-alerts' 

def lambda_handler(event, context):
    started = []
    instances = ec2.instances.filter(Filters=[{'Name': 'tag:AutoSchedule', 'Values': ['true']}])
    
    for instance in instances:
        if instance.state['Name'] == 'stopped':
            instance.start()
            started.append(instance.id)

    if started:
        message = f"The following EC2 instances were started automatically: {', '.join(started)}"
        sns.publish(TopicArn=sns_topic_arn, Message=message, Subject="EC2 Auto-Start Alert")

# /lambda/stop_instance.py
import boto3

ec2 = boto3.resource('ec2')
sns = boto3.client('sns')
sns_topic_arn = 'arn:aws:sns:us-east-1:476114139407:ec2-scheduler-alerts'

def lambda_handler(event, context):
    stopped = []
    instances = ec2.instances.filter(Filters=[{'Name': 'tag:AutoSchedule', 'Values': ['true']}])
    
    for instance in instances:
        if instance.state['Name'] == 'running':
            instance.stop()
            stopped.append(instance.id)

    if stopped:
        message = f"The following EC2 instances were stopped automatically: {', '.join(stopped)}"
        sns.publish(TopicArn=sns_topic_arn, Message=message, Subject="EC2 Auto-Stop Alert")

#SNS Notification
The following EC2 instances were stopped automatically: i-027f7d4d2835bc1df (Time : 11:10)
The following EC2 instances were started automatically: i-027f7d4d2835bc1df (Time : 11:05)

  

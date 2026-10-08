# AWS Lambda Automation with S3 and SNS

## 📌 Project Overview

This project demonstrates a simple serverless AWS workflow using **AWS Lambda, Amazon S3, Amazon SNS, AWS IAM, and Amazon CloudWatch**.

The Lambda function is written in **Python** using the **Boto3 AWS SDK**.

When the Lambda function is executed:

1. Lambda receives the event.
2. A timestamp and execution details are generated.
3. The execution information is stored as a JSON file in Amazon S3.
4. Amazon SNS sends a notification.
5. Lambda returns a successful response.

---

## 🏗️ Architecture

```text
                    ┌────────────────────┐
                    │       User         │
                    │  Invokes Lambda    │
                    └─────────┬──────────┘
                              │
                              ▼
                    ┌────────────────────┐
                    │    AWS Lambda      │
                    │   Python + Boto3   │
                    └─────────┬──────────┘
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    ▼                   ▼
           ┌────────────────┐   ┌────────────────┐
           │   Amazon S3    │   │  Amazon SNS    │
           │                │   │                │
           │ Store JSON     │   │ Send           │
           │ Execution Log  │   │ Notification   │
           └────────────────┘   └───────┬────────┘
                                        │
                                        ▼
                                  ┌────────────┐
                                  │   Email    │
                                  │ Subscriber │
                                  └────────────┘
```

---

## 🔄 Project Flowchart

```text
┌─────────────────────────┐
│          START          │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ Invoke AWS Lambda       │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ Generate Timestamp      │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ Prepare JSON Log Data   │
└────────────┬────────────┘
             │
             ▼
      ┌──────────────┐
      │              │
      ▼              ▼
┌─────────────┐  ┌─────────────┐
│ Upload Log  │  │ Publish SNS │
│ to S3       │  │ Notification│
└──────┬──────┘  └──────┬──────┘
       │                 │
       ▼                 ▼
┌─────────────┐  ┌─────────────┐
│ JSON File   │  │ Email       │
│ Stored in S3│  │ Notification│
└──────┬──────┘  └──────┬──────┘
       │                 │
       └────────┬────────┘
                │
                ▼
      ┌──────────────────┐
      │ Return Success   │
      │ Response         │
      └────────┬─────────┘
               │
               ▼
        ┌─────────────┐
        │     END     │
        └─────────────┘
```

---

## ☁️ AWS Services Used

| AWS Service | Purpose |
|---|---|
| **AWS Lambda** | Executes the Python serverless function |
| **Amazon S3** | Stores Lambda execution logs as JSON files |
| **Amazon SNS** | Sends execution notifications |
| **AWS IAM** | Controls Lambda permissions |
| **Amazon CloudWatch** | Monitors Lambda execution and logs |

---

## 1. AWS Lambda

AWS Lambda is the main compute service used in this project.

The Lambda function:

- Generates a timestamp
- Creates execution log data
- Uploads the log to S3
- Publishes an SNS notification
- Returns a success response

**Runtime:** Python

**SDK:** Boto3

---

## 2. Amazon S3

Amazon S3 is used to store the execution logs.

### Bucket

```text
lambda308
```

### Example object

```text
lambda-logs/
    2026-10-08T09:30:00+00:00-execution.json
```

### Example JSON log

```json
{
    "timestamp": "2026-10-08T09:30:00+00:00",
    "event": {},
    "message": "Lambda function executed successfully"
}
```

---

## 3. Amazon SNS

Amazon SNS is used to send notifications after the Lambda function executes.

### Topic

```text
mytopic
```

### Topic ARN

```text
arn:aws:sns:us-east-1:748861776816:mytopic
```

> For a public GitHub repository, replace the real ARN with a placeholder such as `YOUR_SNS_TOPIC_ARN`.

An email subscription can be added to the SNS topic so that the execution notification is delivered to an email address.

---

## 4. AWS IAM

IAM provides the permissions required by Lambda.

The Lambda execution role requires permissions such as:

```text
s3:PutObject
sns:Publish
```

### S3 resource

```text
arn:aws:s3:::lambda308/*
```

### SNS resource

```text
arn:aws:sns:us-east-1:748861776816:mytopic
```

For production environments, use least-privilege permissions instead of broad administrator access.

---

## 5. Amazon CloudWatch

CloudWatch is used to monitor the Lambda function.

It can be used to view:

- Execution logs
- Errors
- Execution duration
- Invocation count
- Function activity

Navigation:

```text
AWS Console
   ↓
Lambda
   ↓
Your Function
   ↓
Monitor
   ↓
View CloudWatch Logs
```

---

# 🛠️ Technologies Used

| Technology | Purpose |
|---|---|
| Python | Lambda programming language |
| Boto3 | AWS SDK for Python |
| AWS Lambda | Serverless code execution |
| Amazon S3 | Log storage |
| Amazon SNS | Notifications |
| AWS IAM | Access control |
| Amazon CloudWatch | Monitoring |

---

# 🚀 Implementation Process

## Step 1: Create an S3 Bucket

Create an S3 bucket named:

```text
lambda308
```

This bucket stores the Lambda execution logs.

---

## Step 2: Create an SNS Topic

Create an SNS topic named:

```text
mytopic
```

Copy the SNS Topic ARN from the AWS Console.

---

## Step 3: Create an SNS Subscription

Create an email subscription:

```text
Protocol: Email
Endpoint: Your Email Address
```

Confirm the subscription using the email received from AWS SNS.

---

## Step 4: Create the Lambda Function

Create a Lambda function with:

```text
Function Name: LambdaS3SNSFunction
Runtime: Python
```

---

## Step 5: Configure IAM

Give the Lambda execution role the required permissions:

```text
s3:PutObject
sns:Publish
```

---

## Step 6: Add the Python Code

The Lambda function uses Boto3 to communicate with S3 and SNS.

```python
import json
from datetime import datetime, timezone
import boto3

# Initialize AWS clients
s3 = boto3.client("s3")
sns = boto3.client("sns")

# AWS resource configuration
S3_BUCKET_NAME = "lambda308"
SNS_TOPIC_ARN = "YOUR_SNS_TOPIC_ARN"


def lambda_handler(event, context):

    # Get current UTC timestamp
    timestamp = datetime.now(timezone.utc).isoformat()

    # Prepare log data
    log_data = {
        "timestamp": timestamp,
        "event": event,
        "message": "Lambda function executed successfully"
    }

    # Upload log to S3
    s3_key = f"lambda-logs/{timestamp}-execution.json"

    s3.put_object(
        Bucket=S3_BUCKET_NAME,
        Key=s3_key,
        Body=json.dumps(log_data, default=str),
        ContentType="application/json"
    )

    # Send notification using SNS
    message = (
        "Lambda function executed successfully.\n\n"
        f"Timestamp: {timestamp}\n"
        f"S3 Bucket: {S3_BUCKET_NAME}\n"
        f"Log S3 Key: {s3_key}"
    )

    sns.publish(
        TopicArn=SNS_TOPIC_ARN,
        Subject="Lambda Execution Notification",
        Message=message
    )

    return {
        "statusCode": 200,
        "message": "Log stored in S3 and notification sent via SNS.",
        "s3_bucket": S3_BUCKET_NAME,
        "s3_key": s3_key,
        "sns_topic": SNS_TOPIC_ARN
    }
```

---

# 🧪 Testing

Create a test event in the Lambda Console.

Example:

```json
{
    "action": "test"
}
```

Click **Test**.

### Expected result

Lambda should return:

```json
{
    "statusCode": 200,
    "message": "Log stored in S3 and notification sent via SNS."
}
```

---

# ✅ Expected Output

### S3

A JSON execution log should appear under:

```text
lambda-logs/
```

Example:

```text
lambda-logs/2026-10-08T09:30:00+00:00-execution.json
```

### SNS

The SNS topic:

```text
mytopic
```

receives the notification.

### Email

The confirmed SNS email subscriber receives:

```text
Subject: Lambda Execution Notification
```

with the Lambda execution details and S3 log location.

---

# 📁 Suggested Project Structure

```text
AWS-Lambda-S3-SNS/
│
├── lambda_function.py
├── README.md
│
└── screenshots/
    ├── lambda-function.png
    ├── s3-bucket.png
    ├── s3-log.png
    ├── sns-topic.png
    ├── sns-email.png
    ├── iam-policy.png
    └── cloudwatch-logs.png
```

---

# 📸 Screenshots

Add screenshots to the `screenshots` folder and reference them like this:

### Lambda Function

```markdown
![Lambda Function](screenshots/lambda-function.png)
```

### S3 Bucket

```markdown
![S3 Bucket](screenshots/s3-bucket.png)
```

### S3 Execution Log

```markdown
![S3 Log](screenshots/s3-log.png)
```

### SNS Topic

```markdown
![SNS Topic](screenshots/sns-topic.png)
```

### SNS Email Notification

```markdown
![SNS Email](screenshots/sns-email.png)
```

### IAM Permissions

```markdown
![IAM Policy](screenshots/iam-policy.png)
```

### CloudWatch Logs

```markdown
![CloudWatch Logs](screenshots/cloudwatch-logs.png)
```

---

# 🎯 Project Objectives

- Understand AWS serverless computing.
- Learn how to create and configure AWS Lambda functions.
- Use Boto3 to interact with AWS services.
- Store application logs in Amazon S3.
- Send notifications using Amazon SNS.
- Configure IAM permissions.
- Monitor Lambda execution using CloudWatch.
- Understand integration between AWS cloud services.

---

# 🌟 Key Features

- ✅ Serverless architecture
- ✅ Python-based Lambda function
- ✅ Boto3 AWS SDK integration
- ✅ Automatic JSON log generation
- ✅ S3 log storage
- ✅ SNS notifications
- ✅ Email notification support
- ✅ IAM-based access control
- ✅ CloudWatch monitoring

---

# 📚 Key Learnings

Through this project, I learned:

1. How AWS Lambda works as a serverless compute service.
2. How to create and configure an S3 bucket.
3. How to create an SNS topic and subscription.
4. How Lambda communicates with S3 using Boto3.
5. How Lambda publishes messages to SNS.
6. How IAM roles provide secure access to AWS services.
7. How to test and monitor Lambda functions.
8. How multiple AWS services can be integrated into one serverless workflow.

---

# 🔮 Future Enhancements

The project can be extended by adding:

- S3 event triggers
- Scheduled Lambda execution using EventBridge
- DynamoDB for storing execution history
- CloudWatch alarms
- SMS notifications through SNS
- Error and failure notifications
- Multiple SNS subscribers
- Automated monitoring and alerting

---

# 🏁 Conclusion

This project demonstrates a practical **serverless AWS automation workflow** using AWS Lambda, Amazon S3, Amazon SNS, IAM, and CloudWatch.

AWS Lambda performs the main processing, Amazon S3 stores execution logs, and Amazon SNS sends notifications to users. IAM provides controlled access to AWS resources, while CloudWatch helps monitor Lambda execution.

Overall, this project provides hands-on experience in **AWS serverless computing, cloud service integration, Python programming, IAM security, storage, and notification systems**.

---

# 👨‍💻 Author

**Tilak**

## AWS Services

```text
AWS Lambda
Amazon S3
Amazon SNS
AWS IAM
Amazon CloudWatch
```

---
title: "Building Scalable Email Automation with AWS Lambda, SES, and S3"
seoTitle: "Building Scalable Email Automation with AWS Lambda, SES, and S3"
seoDescription: "A Comprehensive Guide to Mass Emailing Using Serverless Architecture"
datePublished: Wed Nov 15 2023 10:44:24 GMT+0000 (Coordinated Universal Time)
cuid: clozmxrci001208l14pqwggdb
slug: building-scalable-email-automation-with-aws-lambda-ses-and-s3
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1706613295936/9b9597e3-9f53-4cb8-bcb3-f2dfdf460388.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1700045643902/71473327-5e2e-4c8b-8e8d-2c93bf34ae11.jpeg
tags: cloud, aws, amazon-web-services, serverless, aws-lambda

---

Welcome, fellow cloud maestros! In this blog post, we'll dive into the exciting world of mass emailing using **AWS Lambda**, **Simple Email Service** (SES), and **Simple Storage Service** (S3). By the end of this project, you'll have a hands-on understanding of serverless functions, cloud storage, and email automation, showcasing your ability to create cost-effective and scalable solutions.

## **Introduction**

Email marketing is a powerful tool for businesses to engage with their audience. However, sending personalized emails to a large audience can be a daunting task. That's where AWS Lambda, SES, and S3 come into play. In this project, I'll walk you through the process of setting up a serverless function that automates the mass emailing process, making it not only efficient but also scalable.

## **1\. Setting Up S3 for CSV File Uploads**

Create a designated S3 bucket where you'll upload a CSV file containing email addresses and message content.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699985843747/eb07995e-921d-4fe3-90dc-e6e0c5986b66.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699985957683/471f9de5-6947-4674-b5d0-d2495d78ff4b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699986160318/e603dae5-2837-4183-8ce9-4caf6fbf2e0f.png align="center")

## **2\. Creating and Verifying SES Identities**

Before diving into the email-sending process, create SES identities for both sender and receiver email addresses. This involves verifying domain ownership and email address verification within the SES console. These identities are essential for maintaining a high deliverability rate and ensuring that your emails reach the intended recipients.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699987017785/1df7e551-1581-44dd-b89c-d3c82e167080.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699987130022/c41a61f4-7594-4239-ad8e-c99276979ded.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699988067572/5092b454-630b-4d35-bba7-7efe8489a867.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699988373373/0a1a62bb-d3d7-4e7e-83af-3545eeb2631b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699988571585/dfc0fd45-493a-4203-8c62-ca2dde4e0eaa.png align="center")

[`firstpirateking.onepiece@gmail.com`](mailto:firstpirateking.onepiece@gmail.com) is the **sender's email address**, and all other email addresses are considered **receiver email addresses**.

When setting up SES identities, you would verify the email address for [`firstpirateking.onepiece@gmail.com`](mailto:firstpirateking.onepiece@gmail.com) as the sender, and for other email addresses, you would ensure their verification within the SES console to maintain a high deliverability rate. This distinction is essential for configuring SES to handle both the sender and receiver roles effectively in your mass email automation project.

## **3\. Setting Up AWS Lambda with SES**

Create an AWS Lambda function and configure it to access SES. **Ensure that your Lambda function has the necessary permissions to send emails through SES**. This step is crucial for establishing a secure and reliable connection between your Lambda function and the email service.

### **3.1 Create a role for Lambda**

To empower your Lambda function, create an IAM (Identity and Access Management) role that grants the required permissions.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699989113397/3556c9bf-b3dd-4299-8ab8-38e9ab21beb8.png align="center")

**Attach Policies:** In the "Attach permissions policies" section, search for and attach the following policies:

* `AmazonS3ReadOnlyAccess`
    
* `AmazonSESFullAccess`
    
* `CloudWatchLogsFullAccess`
    

These policies grant the Lambda function read access to S3, full access to SES, and full access to CloudWatch Logs for monitoring purposes.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699989371947/34559953-27f4-42e5-9ca9-08059c5ec4fd.png align="center")

### **3.2 Create a Lambda Function**

With the IAM role in place, it's time to create the Lambda function. This function will be responsible for handling the logic of your mass email campaign.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699989596090/7d88a57a-bea4-4adb-8101-dc103c565347.png align="center")

## **4\. Setting S3 Event Triggers**

To make the process seamless, configure an S3 event trigger that monitors the designated bucket for new CSV file uploads. Whenever a new file is detected, this trigger will automatically invoke your Lambda function, initiating the email-sending process.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699989703090/2021d58d-f4d0-4ded-8fd7-ad47fd247575.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699989859081/038da2fb-7b29-4f5c-96e2-272aacda945a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699989981216/f4270314-eae3-4cde-8d00-e548a7ac21cc.png align="center")

## **5\. Logic for Lambda Function**

Develop a robust logic within your Lambda function to import the CSV file, extract recipient information, and send personalized messages using AWS SES. This step involves parsing the CSV file, extracting relevant data, and dynamically generating personalized emails for each recipient.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699992981164/aadf2ec1-0276-4b03-b35d-f8f54645fe3e.png align="center")

After writing the code, click `Deploy`

```python
import json
import boto3
import urllib.parse
import csv
from io import StringIO

ses = boto3.client('ses', region_name='ap-south-1')
s3 = boto3.client('s3')

def lambda_handler(event, context):

    s3_bucket_name = event['Records'][0]['s3']['bucket']['name']
    s3_file_key = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'], encoding='utf-8')
    
    print(s3_bucket_name)
    print(s3_file_key)
    
    ses_sender_email = 'firstpirateking.onepiece@gmail.com'
    
    response = s3.get_object(Bucket=s3_bucket_name, Key=s3_file_key)
    csv_data = response['Body'].read().decode('utf-8')
    reader = csv.DictReader(csv_data.splitlines())
    for row in reader:
            recipient_email = row['Email']
            personalized_message = row['Message']
            
            # print(recipient_email)
            # print(personalized_message)
            
            send_email(ses, ses_sender_email, recipient_email, personalized_message)

def send_email(ses, sender_email, recipient_email, message):
    ses.send_email(
	    Source = sender_email,
	    Destination = {
			'ToAddresses': [recipient_email]
	    },
	    Message = {
		    'Subject': {
			    'Data': 'Testing SES',
			    'Charset': 'UTF-8'
		    },
		    'Body': {
			    'Text':{
				    'Data': message,
				    'Charset': 'UTF-8'
			    }
		    }
	    }
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps('Successfully sent email from Lambda using Amazon SES')
    }
```

## **6\. Upload CSV File**

It's time to populate it with your CSV files. Ensure that each file contains crucial information, including recipient email addresses and personalized message content. This step forms the foundation for your mass emailing campaign.

![one piece.csv](https://cdn.hashnode.com/res/hashnode/image/upload/v1699991394831/1ec4f8ef-85dc-4658-9809-6d438837fa55.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699991291683/6046d189-bc79-427d-bc9a-cc6c2b4db31d.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699991305200/a691832e-9910-4a2b-8d0a-01738cffd82f.png align="center")

Once the CSV file is successfully uploaded into the designated S3 bucket, the magic begins. The configured S3 event trigger promptly notifies the `SESLambdaFunction` Lambda function of the new file's presence. This trigger initiates the Lambda function, which, in turn, reads the CSV file, extracts recipient information, and dynamically generates personalized emails. Now, watch as your meticulously crafted emails are sent to the specified email addresses within the CSV file, demonstrating the power of serverless architecture in automating mass email campaigns efficiently.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699991758560/6fcd286e-6cce-41a1-b9d1-bef0d4bb2604.png align="center")

## **Conclusion**

By completing these steps, you've showcased your ability to integrate serverless functions, cloud storage, and email automation – a valuable addition to your skill set and a testament to your capability in the evolving landscape of cloud technology. Whether you're a seasoned developer or a newcomer, this project paves the way for a successful and impactful career in the tech industry.
# 🏗️ Serverless Student Management System (AWS)

This project demonstrates how to build a fully serverless application using **DynamoDB, Lambda, API Gateway, S3, and CloudFront**.  
The system enables inserting and retrieving student data through secure API endpoints integrated with a static web frontend.

---

# ⭐ Summary

- Create DynamoDB Table  
- Create IAM Role for Lambda  
- Lambda Function 1 – Get Students  
- Lambda Function 2 – Insert Student Data  
- Create API Using API Gateway  
- Build Frontend on S3  
- Create CloudFront Distribution  

---

# 📌 Create DynamoDB Table

Create a DynamoDB table:

- **Table Name:** `studentData`  
- **Partition Key:** `studentid` (String)

This table securely stores student records with high availability.

![Image description1](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/x35nhspl6h4lrk0gnqqa.png)

---

# 📌 Create IAM Role for Lambda

- Created a custom IAM role with **AmazonDynamoDBFullAccess**  
- Attached this role to both Lambda functions  
- Enables read/write operations on DynamoDB  

---

# 📌 Lambda Function 1 – Get Students

- Runtime: **Python 3.12**  
- Execution Role: *Existing IAM role with DynamoDB access*  
- Added code from `getstudents.py` and deployed  
- Retrieves all student records from DynamoDB  

```
import json
import boto3

def lambda_handler(event, context):
    dynamodb = boto3.resource('dynamodb', region_name='us-east-1')
    table = dynamodb.Table('studentData')

    response = table.scan()
    data = response['Items']

    while 'LastEvaluatedKey' in response:
        response = table.scan(ExclusiveStartKey=response['LastEvaluatedKey'])
        data.extend(response['Items'])

    return data
```

# 📌 Lambda Function 2 – Insert Student Data

- Runtime: Python 3.12
- Execution Role: Same IAM role
- Added code from insertstudentdata.py and deployed
- Inserts student record into DynamoDB

```
import json
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('studentData')

def lambda_handler(event, context):
    student_id = event['studentid']
    name = event['name']
    student_class = event['class']
    age = event['age']
    
    response = table.put_item(
        Item={
            'studentid': student_id,
            'name': name,
            'class': student_class,
            'age': age
        }
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps('Student data saved successfully!')
    }
```
![Image description2](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gbirrnqdro7ovy67f4uy.png)

# 📌 Create API Using API Gateway

- API Type: REST API (Edge Optimised)
- GET method → Linked to Lambda Function 1
- POST method → Linked to Lambda Function 2
- Enabled CORS
- Deployed and copied the Invoke URL for frontend use

![Image description3](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hw3yu7ycdfoy30guxehv.png)

# 📌 Build Frontend on S3

- Created S3 bucket
- Uploaded index.html and script.js
- Blocked public access (ACL disabled)
- Added GetObject bucket policy for CloudFront
- Enabled Static Website Hosting

**CORS Configuration**
```
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "POST", "PUT", "DELETE", "HEAD"],
    "AllowedOrigins": ["*"],
    "ExposeHeaders": ["ETag"],
    "MaxAgeSeconds": 3000
  }
]
```
# 📌 Create CloudFront Distribution

- Default Root Object → index.html
- Made S3 bucket private
- Updated bucket policy for CloudFront access
- Enables global caching + fast content delivery

![Image description4](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/u7j6pkj6d0jaovpuz1z5.png)

# 📌 OutPut
![Image description6](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/b1hp7mvzy14myiun1tct.png)


![Image description7](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/u6pr9qll2t8en7xwt418.png)


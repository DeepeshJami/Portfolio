# 🚀 AWS Serverless Portfolio Website  
**Hosted on S3 with a Real-Time Visitor Counter**  
*Built with AWS Lambda, DynamoDB, API Gateway & CloudFront*

## 🌟 Features
- **100% Serverless**: No servers to manage (S3 + Lambda).
- **Dynamic Visitor Counter**: Auto-updates using DynamoDB.
- **HTTPS Secure**: CloudFront CDN with free SSL certificate.
- **Custom Domain**: Your own domain (e.g., `deepeshjami.com`).

## 🛠️ Tech Stack
- **AWS**: S3, Lambda, API Gateway, DynamoDB, CloudFront, ACM.
- **Frontend**: HTML, CSS, JavaScript.
- **DNS Management**: Hostinger.

## 📜 Step-by-Step Guide

### 1. Host Static Website on S3
1. **Create S3 Bucket**  
   - Name: `your-portfolio-bucket` (globally unique).
   - Enable *Static Website Hosting* in bucket properties.
   - Upload this `index.html`:
     ```html
     <!DOCTYPE html>
     <html>
     <head>
         <title>My Portfolio</title>
         <style>
             /* Add your CSS here */
         </style>
     </head>
     <body>
         <h1>Welcome to My Portfolio!</h1>
         <p>Visitor Count: <span id="counter">Loading...</span></p>
         <script>
             fetch('YOUR_API_GATEWAY_URL/count')
               .then(response => response.text())
               .then(count => {
                 document.getElementById('counter').textContent = count;
               });
         </script>
     </body>
     </html>
     ```
   - Set bucket policy for public access:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "PublicRead",
                 "Effect": "Allow",
                 "Principal": "*",
                 "Action": "s3:GetObject",
                 "Resource": "arn:aws:s3:::your-portfolio-bucket/*"
             }
         ]
     }
     ```

### 2. Build Visitor Counter Backend
1. **Create DynamoDB Table**  
   - Table name: `VisitorCount`  
   - Partition key: `id` (String).  
   - Add initial item: `{ id: "main_counter", visitor_count: 0 }`.

2. **Create Lambda Function**  
   - Name: `UpdateVisitorCounter`  
   - Runtime: Python 3.12  
   - Code:
     ```python
     import boto3
     def lambda_handler(event, context):
         dynamodb = boto3.resource('dynamodb')
         table = dynamodb.Table('VisitorCount')
         response = table.update_item(
             Key={'id': 'main_counter'},
             UpdateExpression='SET visitor_count = visitor_count + :inc',
             ExpressionAttributeValues={':inc': 1},
             ReturnValues='UPDATED_NEW'
         )
         return {
             'statusCode': 200,
             'headers': { 'Access-Control-Allow-Origin': '*' },
             'body': str(response['Attributes']['visitor_count'])
         }
     ```
   - Attach `AmazonDynamoDBFullAccess` IAM policy.

3. **Create API Gateway**  
   - HTTP API with route `GET /count` linked to Lambda.
   - Deploy API and note the **Invoke URL** (used in `index.html`).

### 3. Set Up CloudFront & Custom Domain
1. **Create CloudFront Distribution**  
   - **Origin Domain**: S3 static website endpoint (e.g., `your-portfolio-bucket.s3-website-us-east-1.amazonaws.com`).  
   - **Alternate Domain Name**: `www.deepeshjami.com`  
   - **SSL Certificate**: Request a free certificate in AWS ACM.

2. **Configure Hostinger DNS**  
   - Add **CNAME Record**:
     ```
     Type: CNAME
     Host: www
     Value: YOUR_CLOUDFRONT_DOMAIN (e.g., d123abc.cloudfront.net)
     ```
   - Add **Redirect Record** (for `deepeshjami.com` → `www.deepeshjami.com`):
     ```
     Type: Redirect
     From: @
     To: https://www.deepeshjami.com
     ```

### 4. Test Your Website!
- Visit `https://www.deepeshjami.com`.  
- Refresh to see the counter increase!

## 🚨 Troubleshooting  
| Issue                | Solution                                  |
|----------------------|-------------------------------------------|
| 403 Forbidden        | Check S3 bucket policy & permissions.     |
| Counter Not Updating | Check Lambda logs in AWS CloudWatch.      |
| SSL Errors           | Ensure ACM certificate is attached to CloudFront. |

## 📂 Project Structure  
├── index.html # Main portfolio page
├── styles/ # CSS files
├── README.md # This file

**License**: [MIT](LICENSE)  

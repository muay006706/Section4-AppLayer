<h1>**Solution** </h1>

<p>Open a terminal session and log in to server using the provided lab credentials: </p>

<code>ssh cloud_user@<PUBLIC_IP_ADDRESS> </code>

<h1>**Create an IAM Role for Lambda** </h1>

1. Change to the directory where the necessary files are located:

<code>cd exercise_files/Section4-AppLayer/Lab1-LambdaS3EventTrigger/ </code>

2. Create an IAM role for Lambda:

<code>aws iam create-role --role-name LambdaIAMRole --description "Lambda Role" --assume-role-policy-document file://lambda_assume_role_policy.json   </code>

3. In the output under "Arn", copy the role ARN and paste it into a text file for later use.

<h1>**Create a Policy for the Lambda Function and Attach It to Role**</h1>

1. Create a Lambda policy:

<code>aws iam create-policy --policy-name LambdaRolePolicy --policy-document file://lambda_execution_policy.json </code>

2. In the output under "Arn", copy the policy ARN for use in the next set of commands.

3. Attach the IAM policy to the IAM role, replacing <POLICY_ARN> with the ARN previously copied:

<code>aws iam attach-role-policy --role-name "LambdaIAMRole" --policy-arn <POLICY_ARN> </code>

<h1>**Create an SNS Topic and Subscribe Your Email Address to It**</h1>

1. Create an SNS topic:
<code>aws sns create-topic --name LambdaTopic --region us-east-1 </code>

2. In the output under "TopicArn", copy the topic ARN for use in the next set of commands.

3. Subscribe an endpoint (e.g., email address) to your topic, replacing <TOPIC_ARN> with the previously copied ARN and <EMAIL_ADDRESS> with your own email:

<code>aws sns subscribe --protocol "email" --topic-arn <TOPIC_ARN> --notification-endpoint <EMAIL_ADDRESS> --region us-east-1 </code>

You should receive the status message "SubscriptionArn": "pending confirmation".

4. To confirm the subscription, access the email previously used, open the SNS email, and click Confirm subscription.

<h1>**Modify the Lambda Function with the SNS Topic ARN and Zip It into a Lambda Deployment Package**</h1>

1. Open the lambda_function file:

<code>vim lambda_function.py </code>

2. To enable sending SNS notifications, uncomment the line client = boto3.client('sns') and the section below:
<html>
   <head>
response = client.publish(
TopicArn='<SNS-TOPIC-ARN>',
Message= payload_str,
Subject='My Lambda S3 event')
<html>
   <head>

3. In TopicArn, replace <SNS-TOPIC-ARN> with the topic ARN previously copied.

4. To save and exit the file, press ESC, type :wq, and press Enter.

5. Zip the file into a deployment package:

<code> zip lambda_function.zip lambda_function.py </code>

<h1>**Create a Lambda Function**</h1>

1. Create a Lambda function, replacing <ROLE_ARN> with the role ARN previously copied into a text file:

<code>aws lambda create-function --memory-size 128 --function-name my-lambda --runtime python3.7 --handler lambda_function.lambda_handler --zip-file fileb://lambda_function.zip --role <ROLE_ARN>
</code>

2. In the output under "FunctionArn", copy the function ARN to a text file for later use.

<h1>**Add Lambda Permission for the S3 Service to /Invoke the Function**</h1>

1. Add Lambda permission, replacing <S3_BUCKET_NAME> with the S3 bucket name provided on the lab credentials page:

<code> aws lambda add-permission --action lambda:InvokeFunction --principal s3.amazonaws.com --statement-id LabS3Trigger --function-name my-lambda --source-arn arn:aws:s3:::<S3_BUCKET_NAME>
</code>

<h1>**Enable and Add Notification Configuration to the S3 Bucket**</h1>

1. Open the bucket-trigger-notification.json file:

<code>vim bucket-trigger-notification.json </code>

2. In the output under "LambdaFunctionArn", delete the existing text and replace it with the function ARN previously copied into a text file:

<code> "LambdaFunctionArn": "<FUNCTION_ARN>" </code>

3. To save and exit the file, press ESC, type :wq, and press Enter.

4. Enable the notification configuration on the S3 website bucket, replacing <S3_BUCKET_NAME> with the bucket name provided on the lab credentials page:

<code> aws s3api put-bucket-notification-configuration --bucket <S3_BUCKET_NAME> --notification-configuration file://bucket-trigger-notification.json  </code>

<h1>**Verify Configuration by Uploading a File to the Provided S3 Bucket**</h1>

1. Upload a file to the bucket, replacing <S3_BUCKET_NAME> with the bucket name provided on the lab page:

<code> aws s3 cp lambda_function.py s3://<S3_BUCKET_NAME> </code>

2. Once successfully uploaded, check your email.

<p>You should receive a notification email with details of the file uploaded to the S3 bucket.</p>
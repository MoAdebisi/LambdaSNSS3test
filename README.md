import boto3
import json

s3 = boto3.client('s3')
sns = boto3.client('sns')

SNS_TOPIC_ARN = 'arn:aws:sns:your-region:123456789012:YourSNSTopic'

def lambda_handler(event, context):
    # Extract bucket and key info from the event
    bucket_name = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    filename = key.split('/')[-1]
    
    # Retrieve file content from S3
    response = s3.get_object(Bucket=bucket_name, Key=key)
    content = response['Body'].read().decode('utf-8')
    
    # Count words
    word_count = len(content.split())
    
    # Format message
    message = f"The word count in the {filename} file is {word_count}."
    subject = "Word Count Result"
    
    # Publish to SNS
    sns.publish(
        TopicArn=SNS_TOPIC_ARN,
        Message=message,
        Subject=subject
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps({'message': message})
    }

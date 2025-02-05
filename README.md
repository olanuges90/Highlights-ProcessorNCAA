# Building NCAA Game Highlights Processor using Docker & AWS ECS, ECR, and AWS Elemental Converter.

## Project Overview

This Project builds a system to automatically process and generate game highlights. RapidAPI is used to obtain game highlights using Docker containers and AWS Elemental Converter to transcode game highlights into multiple resolutions for different platforms.

## Technical Diagram
![highlightsprocessor drawio](https://github.com/user-attachments/assets/91861a62-3b5a-481d-bccb-74edae551671)





## File Overview
- config.py: Handles environment variables for dynamic and flexible configuration management for various environments.  (e.g development, staging and production).
- fetch.py: Fetches NCAA game highlights from RapidAPI and saves them as a JSON file in S3 Bucket as a JSON format(basketball_highlights.json).
- process_one_video.py: Downloads the first video from the JSON file , saves it in the S3 Bucket under the videos folder and logs each step.
- mediaconvert_process.py: Configures MediaConvert jobs for video processing and stores the output in S3.
- run_all.py: Executes all scripts sequentially incorpoerating buffer time between tasks.
- .env: Securely stores environment variables (i.e Variables we don't want to hardcode).
- Dockerfile: Specifies the instrucions for building the Docker image.

## Prerequisities
1. RapidAPI Account: Create an account for accessing NCAA highlights and Images.
   
   API Endpoint: [Sport Highlights API](https://rapidapi.com/highlightly-api-highlightly-api-default/api/sport-highlights-api/playground/apiendpoint_16dd5813-39c6-43f0-aebe-11f891fe5149)

2. Installations Required and verify.
    - Docker: Download and Install Docker desktop, verify with docker --version
    - AWS CLI: Install AWS CLI. Verify Installation: aws --version. Set up AWS Credentials with command: aws configure
    - Python 3: Install Python 3. Verify Installation: python --version
  
3. AWS Account ID: Retrieve from the AWS Management Console.
    - Go to your AWS account, click on your account name in the top right corner and copy your account ID. Keep in a safe place.
4. AWS Credentials: Retrieve ACCESS Key and Secret Key
   - Log in to the AWS Management Console.
   - Go to IAM (Identity and Access Management).
   - Navigate to Users and select your user.
   - Click Security credentials.
   - Under Access keys, click Create access key.
   - Copy and store the Access Key ID and Secret Access Key securely.
## Project Strcuture

     src/
      ├── Dockerfile
      ├── config.py
      ├── fetch.py
      ├── process_one_video.py
      ├── mediaconvert_process.py
      ├── run_all.py
      ├── requirements.txt
      ├── .env
      └── .gitignore
  ## Set up Instructions
  
  ## Step 1: Clone the repository
```sh 
https://github.com/olanuges90/Highlights-ProcessorNCAA
cd src
```
## Step 2: Add API Key to AWS Secrets Manager
```sh
aws secretsmanager create-secret \
--name my-api-key \
--description "API key for accessing the Sport Highlights API" \
--secret-string '{"api_key":"YOUR_ACTUAL_API_KEY"}' \
--region us-east-1
```
## Step 3: Create an IAM role or User
- In your AWS Management console, search for IAM in the search bar and select IAM.
- In the IAM dashboard, click on Roles in the left-hand menu. Click Create role.
- Select S3 as the Use case. Click Next. 
- Under Permissions and Policies, search for AmazonS3FullAccess, MediaConvertFullAccess and AmazonEC2ContainerRegistryFullAccess and click next.
- Under Roles, Enter "Hightlights Processor" as Role Name.
- Click Edit the Trust Policy and Replace it with:

 ```JSON
 {
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": [
          "ec2.amazonaws.com",
          "ecs-tasks.amazonaws.com",
          "mediaconvert.amazonaws.com"
        ],
        "AWS": "arn:aws:iam::<your-account-id>:user/<your-iam-user>"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
 ```

## Step 4: Update the .env File
```bash
RAPIDAPI_KEY="your_api_key"
AWS_ACCESS_KEY_ID="your_aws_access_key_id"
AWS_SECRET_ACCESS_KEY="your_aws_secret_access_key"
S3_BUCKET_NAME="your_s3_bucket_name"
MEDIACONVERT_ENDPOINT="https://your_mediaconvert_endpoint.amazonaws.com"
MEDIACONVERT_ROLE_ARN=arn:"aws:iam::your_account_id:role/HighlightProcessorRole"
```

To find the MediaConvert endpoint, run the command:
```bash
aws mediaconvert describe-endpoints
```
## Step 5: Secure the .env file
```bash
chmod 600 .env
```
## Step 6: Build and Run Docker Container

Build the Docker Image: 
```bash
docker build -t highlight-processor .
```
Run the Docker Container while passing environment variables from a file named .env: 
```bash
docker run --env-file .env highlight-processor
```
This will run the fetch.py, process_one_video.py and mediaconvert_process.py files. 

The following files should be saved in your S3 bucket, Go to your Buckets in your AWS Console and confirm. 
- videos/first_video.mp4
- processed_videos/

<img width="1131" alt="Screenshot 2025-02-04 at 6 19 46 AM" src="https://github.com/user-attachments/assets/784f3aa8-c521-4a03-aad7-d16ed850c38a" />



- Look for a video file named first_video in the videos/ folder within your S3 bucket. Click the video and Open.
  
<img width="770" alt="Screenshot 2025-02-05 at 4 08 23 AM" src="https://github.com/user-attachments/assets/4e0002ad-8328-45e1-88a7-2d7bafc9a04d" />







## Key Takeaways
Docker and AWS Integration.
Identity and Access Management (IAM) and Principle of Least Priviledge.
How to Enhance Media quality using AWS MediaConvert.

## Future Enhancements
Automation of the process using Terraform as IaC.
         





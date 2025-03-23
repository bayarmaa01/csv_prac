# Cloud-Based Test Automation Infrastructure Setup

This guide provides step-by-step instructions for setting up a robust cloud-based test automation infrastructure using AWS and Azure services.

## Prerequisites

- AWS account with administrator access
- Azure account with subscription
- Basic knowledge of cloud concepts
- Git and GitHub account
- SSH key pair for secure connections

## AWS Infrastructure Setup

### Step 1: Set Up IAM Users and Permissions

1. Log in to the AWS Management Console
2. Navigate to IAM (Identity and Access Management)
3. Create a new IAM group for test automation:
   ```
   Group Name: TestAutomationGroup
   ```
4. Attach the following policies to the group:
   - AmazonEC2FullAccess
   - AmazonS3FullAccess
   - AmazonRDSFullAccess
   - AWSLambdaFullAccess
   - AmazonECR-FullAccess

5. Create a new IAM user for automation:
   ```
   Username: test-automation-user
   Access type: Programmatic access
   ```
6. Add the user to the TestAutomationGroup
7. Save the Access Key ID and Secret Access Key securely

### Step 2: Configure AWS CLI

1. Install AWS CLI:
   ```bash
   # Windows (using PowerShell as administrator)
   msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi

   # macOS
   curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
   sudo installer -pkg AWSCLIV2.pkg -target /

   # Linux
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   sudo ./aws/install
   ```

2. Configure AWS CLI with your credentials:
   ```bash
   aws configure
   ```
   Enter your Access Key ID, Secret Access Key, default region (e.g., us-west-2), and output format (json)

### Step 3: Set Up Amazon S3 for Test Artifacts

1. Create an S3 bucket for test artifacts:
   ```bash
   aws s3 mb s3://test-automation-artifacts-[your-unique-id]
   ```

2. Enable versioning on the bucket:
   ```bash
   aws s3api put-bucket-versioning --bucket test-automation-artifacts-[your-unique-id] --versioning-configuration Status=Enabled
   ```

3. Create folders for different test types:
   ```bash
   aws s3api put-object --bucket test-automation-artifacts-[your-unique-id] --key ui-tests/
   aws s3api put-object --bucket test-automation-artifacts-[your-unique-id] --key api-tests/
   aws s3api put-object --bucket test-automation-artifacts-[your-unique-id] --key performance-tests/
   aws s3api put-object --bucket test-automation-artifacts-[your-unique-id] --key security-tests/
   ```

### Step 4: Create EC2 Instances for Test Execution

1. Create a security group for test automation:
   ```bash
   aws ec2 create-security-group --group-name TestAutomationSG --description "Security group for test automation EC2 instances"
   ```

2. Add inbound rules to the security group:
   ```bash
   aws ec2 authorize-security-group-ingress --group-name TestAutomationSG --protocol tcp --port 22 --cidr 0.0.0.0/0
   aws ec2 authorize-security-group-ingress --group-name TestAutomationSG --protocol tcp --port 80 --cidr 0.0.0.0/0
   aws ec2 authorize-security-group-ingress --group-name TestAutomationSG --protocol tcp --port 443 --cidr 0.0.0.0/0
   ```

3. Create an EC2 instance:
   ```bash
   aws ec2 run-instances --image-id ami-0c55b159cbfafe1f0 --count 1 --instance-type t2.large --key-name your-key-pair --security-group-ids TestAutomationSG --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=TestAutomationInstance}]'
   ```
   Note: Replace `ami-0c55b159cbfafe1f0` with the latest Amazon Linux 2 AMI ID for your region

4. Create an initialization script `init_test_env.sh`:
   ```bash
   #!/bin/bash
   yum update -y
   yum install -y docker git jq
   systemctl start docker
   systemctl enable docker
   usermod -aG docker ec2-user
   curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   chmod +x /usr/local/bin/docker-compose
   pip3 install pytest selenium appium requests allure-pytest
   ```

5. Copy the script to the EC2 instance:
   ```bash
   aws ec2 describe-instances --filters "Name=tag:Name,Values=TestAutomationInstance" --query "Reservations[*].Instances[*].PublicDnsName" --output text
   scp -i /path/to/your-key-pair.pem init_test_env.sh ec2-user@[EC2-PUBLIC-DNS]:~/
   ssh -i /path/to/your-key-pair.pem ec2-user@[EC2-PUBLIC-DNS] "chmod +x ~/init_test_env.sh && sudo ~/init_test_env.sh"
   ```

### Step 5: Set Up Amazon ECR for Docker Images

1. Create an ECR repository:
   ```bash
   aws ecr create-repository --repository-name test-automation-images
   ```

2. Log in to ECR:
   ```bash
   aws ecr get-login-password --region [your-region] | docker login --username AWS --password-stdin [your-account-id].dkr.ecr.[your-region].amazonaws.com
   ```

3. Create a Dockerfile for Selenium Grid:
   ```dockerfile
   FROM selenium/hub:4.8.0
   LABEL maintainer="Your Name <your.email@example.com>"
   ```

4. Build and push the Docker image:
   ```bash
   docker build -t test-automation-images:selenium-hub .
   docker tag test-automation-images:selenium-hub [your-account-id].dkr.ecr.[your-region].amazonaws.com/test-automation-images:selenium-hub
   docker push [your-account-id].dkr.ecr.[your-region].amazonaws.com/test-automation-images:selenium-hub
   ```

## Azure Infrastructure Setup

### Step 1: Install Azure CLI

1. Install Azure CLI:
   ```bash
   # Windows (PowerShell)
   Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi
   Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'

   # macOS
   brew update && brew install azure-cli

   # Linux
   curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
   ```

2. Log in to Azure:
   ```bash
   az login
   ```

### Step 2: Create Azure Resource Group

1. Create a resource group for test automation:
   ```bash
   az group create --name TestAutomationRG --location eastus
   ```

### Step 3: Set Up Azure App Service

1. Create an App Service Plan:
   ```bash
   az appservice plan create --name TestAutomationPlan --resource-group TestAutomationRG --sku S1 --is-linux
   ```

2. Create a Web App for your test dashboard:
   ```bash
   az webapp create --resource-group TestAutomationRG --plan TestAutomationPlan --name test-automation-dashboard-[unique-name] --runtime "NODE|14-lts"
   ```

3. Configure deployment from GitHub:
   ```bash
   az webapp deployment source config --name test-automation-dashboard-[unique-name] --resource-group TestAutomationRG --repo-url https://github.com/yourusername/test-dashboard --branch main --git-token [your-github-token]
   ```

### Step 4: Set Up Azure DevOps Pipeline

1. Create a new Azure DevOps organization or use an existing one
2. Create a new project:
   ```
   Project Name: TestAutomation
   Visibility: Private
   Version control: Git
   ```

3. Import your test repository from GitHub or create a new one

4. Create a new pipeline using the following YAML:
   ```yaml
   trigger:
   - main

   pool:
     vmImage: 'ubuntu-latest'

   steps:
   - task: UsePythonVersion@0
     inputs:
       versionSpec: '3.10'
       addToPath: true

   - script: |
       python -m pip install --upgrade pip
       pip install pytest selenium requests allure-pytest
     displayName: 'Install dependencies'

   - script: |
       pytest tests/ --alluredir=$(Build.ArtifactStagingDirectory)/allure-results
     displayName: 'Run tests'

   - task: PublishTestResults@2
     inputs:
       testResultsFormat: 'JUnit'
       testResultsFiles: '**/TEST-*.xml'
       mergeTestResults: true
       testRunTitle: 'Test Results'

   - task: PublishBuildArtifacts@1
     inputs:
       pathtoPublish: '$(Build.ArtifactStagingDirectory)/allure-results'
       artifactName: 'test-results'
       publishLocation: 'Container'
   ```

### Step 5: Set Up Azure Key Vault for Secrets

1. Create an Azure Key Vault:
   ```bash
   az keyvault create --name test-automation-keyvault --resource-group TestAutomationRG --location eastus
   ```

2. Add secrets for test environment:
   ```bash
   az keyvault secret set --vault-name test-automation-keyvault --name TestDBPassword --value "your-secure-password"
   az keyvault secret set --vault-name test-automation-keyvault --name TestAPIKey --value "your-api-key"
   ```

3. Configure your Azure DevOps pipeline to access Key Vault:
   ```yaml
   - task: AzureKeyVault@2
     inputs:
       azureSubscription: 'Your-Azure-Service-Connection'
       KeyVaultName: 'test-automation-keyvault'
       SecretsFilter: 'TestDBPassword,TestAPIKey'
       RunAsPreJob: true
   ```

## Integration Between AWS and Azure

### Step 1: Set Up Cross-Cloud Authentication

1. Create an Azure service principal:
   ```bash
   az ad sp create-for-rbac --name TestAutomationSP --role Contributor --scopes /subscriptions/[your-subscription-id]/resourceGroups/TestAutomationRG
   ```
   Save the appId, displayName, password, and tenant

2. Store Azure credentials in AWS Secrets Manager:
   ```bash
   aws secretsmanager create-secret --name AzureServicePrincipal --secret-string '{"appId":"[your-app-id]","password":"[your-password]","tenant":"[your-tenant-id]"}'
   ```

### Step 2: Create Data Flow Between Clouds

1. Create an AWS Lambda function for cross-cloud data sync:
   ```bash
   # Create a file named azure_sync.py
   import boto3
   import json
   import requests
   import os

   def lambda_handler(event, context):
       # Get Azure credentials from Secrets Manager
       client = boto3.client('secretsmanager')
       azure_creds = json.loads(client.get_secret_value(SecretId='AzureServicePrincipal')['SecretString'])
       
       # Get Azure token
       token_url = f"https://login.microsoftonline.com/{azure_creds['tenant']}/oauth2/token"
       token_data = {
           'grant_type': 'client_credentials',
           'client_id': azure_creds['appId'],
           'client_secret': azure_creds['password'],
           'resource': 'https://management.azure.com/'
       }
       token_r = requests.post(token_url, data=token_data)
       token = token_r.json().get('access_token')
       
       # Get test results from S3
       s3 = boto3.client('s3')
       test_results = s3.get_object(Bucket='test-automation-artifacts-[your-unique-id]', Key='latest-results.json')
       result_data = json.loads(test_results['Body'].read().decode('utf-8'))
       
       # Send to Azure Storage
       headers = {
           'Authorization': f'Bearer {token}',
           'Content-Type': 'application/json'
       }
       azure_url = f"https://test-automation-dashboard-[unique-name].azurewebsites.net/api/results"
       r = requests.post(azure_url, headers=headers, json=result_data)
       
       return {
           'statusCode': r.status_code,
           'body': r.text
       }
   ```

2. Create the Lambda function:
   ```bash
   zip azure_sync.zip azure_sync.py
   aws lambda create-function --function-name AzureTestResultsSync --runtime python3.9 --role [your-lambda-execution-role-arn] --handler azure_sync.lambda_handler --zip-file fileb://azure_sync.zip
   ```

3. Set up an S3 event trigger for the Lambda function:
   ```bash
   aws lambda add-permission --function-name AzureTestResultsSync --statement-id s3-trigger --action lambda:InvokeFunction --principal s3.amazonaws.com --source-arn arn:aws:s3:::test-automation-artifacts-[your-unique-id] --source-account [your-account-id]
   
   aws s3api put-bucket-notification-configuration --bucket test-automation-artifacts-[your-unique-id] --notification-configuration '{
     "LambdaFunctionConfigurations": [
       {
         "LambdaFunctionArn": "arn:aws:lambda:[your-region]:[your-account-id]:function:AzureTestResultsSync",
         "Events": ["s3:ObjectCreated:*"],
         "Filter": {
           "Key": {
             "FilterRules": [
               {
                 "Name": "suffix",
                 "Value": "results.json"
               }
             ]
           }
         }
       }
     ]
   }'
   ```

## Automation Framework Setup

### Step 1: Create a Test Repository Structure

Create a GitHub repository with the following structure:

```
test-automation/
├── .github/
│   └── workflows/
│       └── ci.yml
├── conftest.py
├── requirements.txt
├── infra/
│   ├── aws/
│   │   ├── setup_aws.py
│   │   └── teardown_aws.py
│   ├── azure/
│   │   ├── setup_azure.py
│   │   └── teardown_azure.py
│   └── docker/
│       └── docker-compose.yml
├── tests/
│   ├── ui/
│   │   ├── conftest.py
│   │   └── test_ui.py
│   ├── api/
│   │   ├── conftest.py
│   │   └── test_api.py
│   └── performance/
│       ├── conftest.py
│       └── test_performance.py
└── utils/
    ├── aws_utils.py
    ├── azure_utils.py
    └── reporting.py
```

### Step 2: Create Infrastructure as Code Templates

1. Create AWS CloudFormation template `aws_template.yaml`:
   ```yaml
   AWSTemplateFormatVersion: '2010-09-09'
   Resources:
     TestVPC:
       Type: AWS::EC2::VPC
       Properties:
         CidrBlock: 10.0.0.0/16
         EnableDnsSupport: true
         EnableDnsHostnames: true
         Tags:
           - Key: Name
             Value: TestAutomationVPC
     
     TestSubnet:
       Type: AWS::EC2::Subnet
       Properties:
         VpcId: !Ref TestVPC
         CidrBlock: 10.0.1.0/24
         MapPublicIpOnLaunch: true
         AvailabilityZone: !Select [0, !GetAZs ""]
         Tags:
           - Key: Name
             Value: TestAutomationSubnet
     
     TestSecurityGroup:
       Type: AWS::EC2::SecurityGroup
       Properties:
         GroupDescription: Security group for test automation
         VpcId: !Ref TestVPC
         SecurityGroupIngress:
           - IpProtocol: tcp
             FromPort: 22
             ToPort: 22
             CidrIp: 0.0.0.0/0
           - IpProtocol: tcp
             FromPort: 80
             ToPort: 80
             CidrIp: 0.0.0.0/0
           - IpProtocol: tcp
             FromPort: 443
             ToPort: 443
             CidrIp: 0.0.0.0/0
     
     TestInstance:
       Type: AWS::EC2::Instance
       Properties:
         InstanceType: t2.large
         ImageId: ami-0c55b159cbfafe1f0
         KeyName: your-key-pair
         SubnetId: !Ref TestSubnet
         SecurityGroupIds:
           - !Ref TestSecurityGroup
         Tags:
           - Key: Name
             Value: TestAutomationInstance
         UserData:
           Fn::Base64: !Sub |
             #!/bin/bash
             yum update -y
             yum install -y docker git jq
             systemctl start docker
             systemctl enable docker
             usermod -aG docker ec2-user
   ```

2. Create Azure ARM template `azure_template.json`:
   ```json
   {
     "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
     "contentVersion": "1.0.0.0",
     "parameters": {
       "webAppName": {
         "type": "string",
         "defaultValue": "test-automation-dashboard",
         "metadata": {
           "description": "Name of the Web App"
         }
       },
       "location": {
         "type": "string",
         "defaultValue": "eastus",
         "metadata": {
           "description": "Location for resources"
         }
       }
     },
     "resources": [
       {
         "type": "Microsoft.Web/serverfarms",
         "apiVersion": "2020-06-01",
         "name": "TestAutomationPlan",
         "location": "[parameters('location')]",
         "sku": {
           "name": "S1",
           "tier": "Standard"
         },
         "properties": {
           "reserved": true
         },
         "kind": "linux"
       },
       {
         "type": "Microsoft.Web/sites",
         "apiVersion": "2020-06-01",
         "name": "[parameters('webAppName')]",
         "location": "[parameters('location')]",
         "dependsOn": [
           "TestAutomationPlan"
         ],
         "properties": {
           "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', 'TestAutomationPlan')]",
           "siteConfig": {
             "linuxFxVersion": "NODE|14-lts"
           }
         }
       }
     ]
   }
   ```

### Step 3: Set Up Docker-Compose for Local Testing

Create `docker-compose.yml`:
```yaml
version: '3'
services:
  selenium-hub:
    image: selenium/hub:4.8.0
    container_name: selenium-hub
    ports:
      - "4444:4444"

  chrome:
    image: selenium/node-chrome:4.8.0
    volumes:
      - /dev/shm:/dev/shm
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
    ports:
      - "5900:5900"

  firefox:
    image: selenium/node-firefox:4.8.0
    volumes:
      - /dev/shm:/dev/shm
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
    ports:
      - "5901:5900"

  test-runner:
    image: python:3.10
    volumes:
      - ./:/app
    working_dir: /app
    depends_on:
      - chrome
      - firefox
    command: sh -c "pip install -r requirements.txt && pytest tests/ -v"
```

## CI/CD Pipeline Setup

### Step 1: Create GitHub Actions Workflow

Create `.github/workflows/ci.yml`:
```yaml
name: Test Automation CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 0 * * *'  # Run daily at midnight

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.10'
    
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-west-2
    
    - name: Configure Azure credentials
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    
    - name: Set up infrastructure
      run: |
        python infra/aws/setup_aws.py
        python infra/azure/setup_azure.py
    
    - name: Run tests
      run: |
        pytest tests/ --alluredir=./allure-results
    
    - name: Upload test results to S3
      run: |
        aws s3 cp ./allure-results s3://test-automation-artifacts-[your-unique-id]/allure-results/ --recursive
    
    - name: Generate Allure report
      uses: simple-elf/allure-report-action@master
      if: always()
      with:
        allure_results: allure-results
        allure_history: allure-history
    
    - name: Publish Test Report
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: test-report
        path: allure-report/
    
    - name: Tear down infrastructure
      if: always()
      run: |
        python infra/aws/teardown_aws.py
        python infra/azure/teardown_azure.py
```

## Troubleshooting Common Issues

### AWS Connectivity Issues
- Check security group configurations
- Verify IAM permissions
- Inspect EC2 instance status and logs: `aws ec2 get-console-output --instance-id [YOUR_INSTANCE_ID]`

### Azure Deployment Failures
- Check Azure Activity Log in the portal
- Verify service principal permissions
- Check resource group deployment history: `az group deployment list --resource-group TestAutomationRG`

### Cross-Cloud Integration Issues
- Verify network connectivity between AWS and Azure
- Check Lambda function execution logs
- Verify Azure service principal credentials in AWS Secrets Manager

## Best Practices

1. Use infrastructure as code for all cloud resources
2. Store all secrets in secure vaults (AWS Secrets Manager or Azure Key Vault)
3. Use Docker containers for consistent test environments
4. Implement comprehensive logging and monitoring
5. Set up automated cleanup of test resources to control costs
6. Use tagging to track resource costs and ownership
7. Implement retry mechanisms for flaky tests
8. Set up alerting for test failures

# AmazonConnect-AI-Agents-ListSpansViewer
Web Based Viewer for ListSpansAPI
# ListSpans Viewer for Amazon Connect AI Agents

[![AWS CloudFormation](https://img.shields.io/badge/AWS-CloudFormation-orange?logo=amazon-aws)](https://aws.amazon.com/cloudformation/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/badge/version-5.1.0-blue.svg)](https://github.com/yourusername/listspans-viewer/releases)

A web-based visualization tool for the Amazon Connect AI Agents [ListSpans API](https://docs.aws.amazon.com/amazon-q-connect/latest/APIReference/API_ListSpans.html). Deploy with a single CloudFormation template and instantly start debugging and analyzing your AI agent sessions.

![ListSpans Viewer Screenshot](docs/screenshot.png)

## 🎯 What is This?

When building AI agents with Amazon Connect, debugging agent behavior can be challenging. The ListSpans API provides detailed telemetry about what happened during an agent session, including:

- **Inference calls** - LLM invocations with input/output messages
- **Tool executions** - Which tools were called and their results
- **Agent invocations** - Sub-agent calls and orchestration
- **Token usage** - Input/output token counts per call
- **Timing data** - Duration of each operation
- **Error details** - What went wrong and where

This tool provides a clean, visual interface to explore this data without writing code or using the AWS CLI.

## ✨ Features

- 🚀 **One-Click Deploy** - Single CloudFormation template deploys everything
- 🔐 **Secure** - Uses IAM roles, no credentials in the browser
- 🎨 **Visual Interface** - Color-coded spans, expandable details, syntax highlighting
- 🔍 **Filtering** - Filter by span type (inference, tool, agent) or errors
- 📊 **Statistics** - Token counts, success/error rates at a glance
- 📋 **Export** - Download raw JSON or copy to clipboard
- 🌐 **Multi-Region** - Query any supported AWS region
- ⚡ **Serverless** - No servers to manage, scales automatically

## 🏗️ Architecture

┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ │ CloudFront │────▶│ S3 Bucket │ │ Amazon Connect │ │ Distribution │ │ (index.html) │ │ AI Agents │ └────────┬────────┘ └─────────────────┘ └────────▲────────┘ │ │ │ API calls │ ListSpans ▼ │ ┌─────────────────┐ ┌─────────────────┐ │ │ API Gateway │────▶│ Lambda │──────────────┘ │ (HTTP API) │ │ (Python 3.12) │ └─────────────────┘ └─────────────────┘


**Components:**
- **CloudFront** - CDN for fast, secure content delivery
- **S3** - Hosts the static web application
- **API Gateway** - HTTP API endpoint for the Lambda function
- **Lambda** - Signs requests and calls the ListSpans API

## 🚀 Quick Start

### Prerequisites

- AWS Account with appropriate permissions
- Amazon Connect AI Agents instance with sessions to analyze
- AWS CLI configured (optional, for CLI deployment)

### Deploy via AWS Console

1. Click the button below to launch the stack:

   [![Launch Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home#/stacks/new?stackName=listspans-viewer&templateURL=https://your-bucket.s3.amazonaws.com/template.yaml)

2. Or manually:
   - Go to [CloudFormation Console](https://console.aws.amazon.com/cloudformation)
   - Click **Create stack** → **With new resources**
   - Upload the `template.yaml` file
   - Enter a stack name (e.g., `listspans-viewer`)
   - Configure parameters (optional)
   - Check "I acknowledge that AWS CloudFormation might create IAM resources"
   - Click **Create stack**

3. Wait for stack creation (~3-5 minutes)

4. Find the **WebsiteURL** in the Outputs tab

### Deploy via AWS CLI

```bash
# Clone the repository
git clone https://github.com/yourusername/listspans-viewer.git
cd listspans-viewer

# Deploy the stack
aws cloudformation create-stack \
  --stack-name listspans-viewer \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_IAM

# Wait for completion
aws cloudformation wait stack-create-complete --stack-name listspans-viewer

# Get the website URL
aws cloudformation describe-stacks \
  --stack-name listspans-viewer \
  --query 'Stacks[0].Outputs[?OutputKey==`WebsiteURL`].OutputValue' \
  --output text

📖 Usage

    Open the Website URL from CloudFormation outputs

    Enter Query Parameters:
        Region - AWS region where your AI agent is deployed
        Assistant ID - Your Amazon Connect AI Agent assistant ID
        Session ID - The session ID you want to analyze
        Max Results - Number of spans to retrieve (1-1000)

    Click "Fetch Spans" to retrieve the data

    Explore the Results:
        Click on any span card to expand details
        Use filter chips to show specific span types
        View input/output messages, tool calls, and raw attributes
        Check the statistics bar for quick insights

    Export Data:
        Export JSON - Download the raw response as a file
        Copy Raw - Copy the JSON to clipboard


🔍 Finding Your Assistant ID and Session ID

To use the ListSpans Viewer, you need two pieces of information:

    Assistant ID - Identifies your Amazon Connect AI Agent
    Session ID - Identifies a specific conversation/call session

Method 1: From Contact Flow Logs (CloudWatch)

The easiest way to find these IDs is from your Amazon Connect contact flow logs in CloudWatch.

Step 1: Enable Contact Flow Logging

Ensure your contact flow has logging enabled (Set logging behavior block).

Step 2: Find the Logs in CloudWatch

    Go to CloudWatch → Log groups
    Find your Connect instance log group: /aws/connect/<instance-name>
    Search for your contact using the Contact ID or timestamp

Step 3: Find the Assistant ID

Look for the SetWisdomAssistant log entry:

{
    "ContactId": "e53230fc-855d-4724-ad7f-7b024c64444b",
    "ContactFlowModuleType": "SetWisdomAssistant",
    "Identifier": "Create Connect Assistant session",
    "Parameters": {
        "WisdomAssistantArn": "arn:aws:wisdom:us-west-2:445149420241:assistant/4738423e-4752-4a2a-9593-bfe4b21e1635"
    }
}

The Assistant ID is the UUID after /assistant/:

4738423e-4752-4a2a-9593-bfe4b21e1635

Step 4: Find the Session ID

Look for the SetContactData log entry that contains WisdomSessionArn:

{
    "ContactId": "e53230fc-855d-4724-ad7f-7b024c64444b",
    "ContactFlowModuleType": "SetContactData",
    "Identifier": "Create Connect Assistant session-y2aiuEqbZB",
    "Parameters": {
        "WisdomSessionArn": "arn:aws:wisdom:us-west-2:445149420241:session/4738423e-4752-4a2a-9593-bfe4b21e1635/c976d916-98ba-46d8-a82a-7e8dd493e94e"
    }
}

The Session ID is the UUID after /session/<assistant-id>/:

c976d916-98ba-46d8-a82a-7e8dd493e94e

🔧 Configuration

CloudFormation Parameters

Parameter
	

Default
	

Description

ProjectName
	

listspans-viewer
	

Prefix for all resource names

DefaultAssistantId
	

(empty)
	

Pre-populated Assistant ID in the UI

Customizing the Default Assistant ID

aws cloudformation update-stack \
  --stack-name listspans-viewer \
  --template-body file://template.yaml \
  --parameters ParameterKey=DefaultAssistantId,ParameterValue=your-assistant-id \
  --capabilities CAPABILITY_IAM

🔒 Security

    No credentials in browser - The Lambda function uses IAM roles to authenticate
    HTTPS only - CloudFront enforces TLS
    Private S3 bucket - Origin Access Control restricts direct S3 access
    Minimal permissions - Lambda role only has wisdom:* and qconnect:* permissions

IAM Permissions Required for Deployment

The deploying user/role needs:

    cloudformation:* - Stack management
    s3:* - Bucket creation and management
    lambda:* - Function creation
    apigateway:* - API Gateway setup
    cloudfront:* - Distribution creation
    iam:* - Role creation (with CAPABILITY_IAM)

🐛 Troubleshooting

"No spans found for this session"

    Verify the session ID is correct
    Ensure the session has completed at least one turn
    Check that the region matches where the agent is deployed

"AccessDeniedException"

    The Lambda execution role may not have permission to call ListSpans
    Verify the assistant ID is in the same AWS account

"ResourceNotFoundException"

    The assistant ID or session ID doesn't exist
    Double-check both IDs and the region

Stale Content After Update

Clear the CloudFront cache:

aws cloudfront create-invalidation \
  --distribution-id YOUR_DIST_ID \
  --paths "/*"

Or wait for the cache to expire (~24 hours) and hard refresh your browser.

🧪 Testing the Lambda Directly

aws lambda invoke \
  --function-name listspans-viewer-list-spans \
  --payload '{
    "requestContext": {"http": {"method": "POST"}},
    "body": "{\"region\":\"us-west-2\",\"assistantId\":\"YOUR_ASSISTANT_ID\",\"sessionId\":\"YOUR_SESSION_ID\",\"maxResults\":100}"
  }' \
  --cli-binary-format raw-in-base64-out \
  response.json

cat response.json | jq .

📁 Project Structure

listspans-viewer/
├── template.yaml          # CloudFormation template (all-in-one)
├── README.md              # This file
├── LICENSE                # MIT License
└── docs/
    └── screenshot.png     # UI screenshot

🔄 Updating

To update to a new version:

aws cloudformation update-stack \
  --stack-name listspans-viewer \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_IAM

🗑️ Cleanup

To remove all resources:

# Empty the S3 bucket first (required)
BUCKET=$(aws cloudformation describe-stacks \
  --stack-name listspans-viewer \
  --query 'Stacks[0].Outputs[?OutputKey==`S3Bucket`].OutputValue' \
  --output text)

aws s3 rm s3://$BUCKET --recursive

# Delete the stack
aws cloudformation delete-stack --stack-name listspans-viewer

🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

    Fork the repository
    Create your feature branch (git checkout -b feature/AmazingFeature)
    Commit your changes (git commit -m 'Add some AmazingFeature')
    Push to the branch (git push origin feature/AmazingFeature)
    Open a Pull Request

📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

🙏 Acknowledgments

    Built for debugging Amazon Connect AI Agents
    Uses the ListSpans API

📫 Support

    🐛 Report a bug
    💡 Request a feature
    📖 Amazon Connect AI Agents Documentation

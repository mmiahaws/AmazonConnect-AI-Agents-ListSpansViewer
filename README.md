# ListSpans Viewer

A web-based viewer for Amazon Connect AI Agent sessions. Browse, replay, and debug conversations between customers and AI Agents powered by Amazon Connect AI Agents.

![Transcript View](screenshots/sequence-diagram.png)

## What It Does

ListSpans Viewer connects to your Amazon Connect instance and lets you:

- **Browse Sessions** — Scan CloudWatch Logs to discover recent AI Agent conversations
- **View Timeline** — See all conversation events (spans) in chronological order
- **Read Transcripts** — Chat-style view showing customer messages and AI responses with clear badges
- **Sequence Diagrams** — Visual flow of the conversation between customer, AI agent, and tools
- **Debug Tools** — Inspect every tool execution, its input/output, and duration
- **Track Errors** — Identify failed spans and troubleshoot AI Agent issues

![Session Timeline](screenshots/timeline-view.png)

## Architecture

```
User Browser
    ↓
CloudFront (HTTPS)
    ↓
S3 Bucket (Private, OAC)
    ↓
API Gateway (HTTP API)
    ↓
Lambda Functions
    ├── GetConfig    → Returns deployment settings
    ├── ListSpans    → Fetches AI Agent spans via Wisdom API
    └── BrowseLogs   → Scans CloudWatch Logs for sessions
        ↓
    ┌───┴───┐
    ↓       ↓
ListSpansAPI  CloudWatch Logs
```

Everything deploys via a single CloudFormation template. No servers to manage.

## Prerequisites

- An AWS account with appropriate permissions
- Amazon Connect instance with AI Agent enabled
- CloudWatch Logs enabled for your Connect instance
- AWS CLI installed and configured (or use AWS CloudShell)

### Required IAM Permissions

Your deploying user/role needs permissions to create:
- S3 buckets and bucket policies
- CloudFront distributions
- Lambda functions
- API Gateway (HTTP API)
- IAM roles

## What You'll Need

Before deploying, gather these values:

| Value | Format | Where to Find |
|-------|--------|---------------|
| Assistant ID | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` | Connect console → AI Agents → Your Agent → Assistant ID |
| Log Group | `/aws/connect/YOUR_INSTANCE_ALIAS` | Connect console → Your instance → Note the alias |
| Region | e.g. `us-west-2`, `eu-west-2` | The region where your Connect instance lives |
| Connect Instance ID | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` | Connect console → Your instance (optional) |

## Deployment

### Option 1: AWS CloudShell 

1. Download the zip from this repo: `listspans-viewer-v2-deployment.zip`
2. Open [AWS CloudShell](https://console.aws.amazon.com/cloudshell/) in the region where your Connect instance lives
3. Upload the zip: click **Actions → Upload file**
4. Run:

```bash
unzip listspans-viewer-v2-deployment.zip
cd deployment-package
chmod +x deploy.sh cleanup.sh
./deploy.sh
```

5. Follow the prompts — enter your Assistant ID, Log Group, and region
6. The script outputs your CloudFront URL when done

![CloudShell Deployment](screenshots/cloudshell-deploy.png)

### Option 2: Mac / Linux

```bash
# Clone the repo
git clone https://gitlab.aws.dev/mmiahaws/ListSpansViewer.git
cd ListSpansViewer/deployment-package

# Make scripts executable
chmod +x deploy.sh cleanup.sh

# Deploy
./deploy.sh
```

The script prompts for all required values. Defaults are provided where possible — just press Enter to accept them.

**Prompt values:**
- Stack Name → lowercase, hyphens only (e.g. `listspans-viewer`)
- Region → where your Connect instance is (e.g. `eu-west-2`)
- Assistant ID → UUID from Connect console
- Log Group → `/aws/connect/your-instance-alias`
- Instance ID → optional, press Enter to skip
- Project Name → lowercase, hyphens only (e.g. `listspans-viewer`)

### Option 3: Windows (PowerShell)

```powershell
# Clone the repo
git clone https://gitlab.aws.dev/mmiahaws/ListSpansViewer.git
cd ListSpansViewer\deployment-package

# Deploy
.\deploy.ps1
```

**Windows requirements:**
- PowerShell 5.1+ (built into Windows 10/11)
- AWS CLI installed — [download here](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- AWS credentials configured (`aws configure`)

The PowerShell script has the same prompts as the bash version.


## Usage

Once deployed, open the CloudFront URL in your browser.

### Browse Sessions

1. Click **Browse Logs** on the main page
2. Set the time range (default: 24 hours)
3. Click **Scan Logs** to find sessions
4. Click any session to load its spans

![Browse Sessions](screenshots/browse-sessions.png)

### View a Session

Enter a Session ID directly, or pick one from Browse Logs. The viewer shows:

- **Timeline** — chronological list of all spans with duration and status
- **Transcript** — customer and AI messages in a chat-style layout
- **Sequence Diagram** — visual flow between participants
- **Tool Details** — expand any tool span to see inputs/outputs


![Transcript View](screenshots/transcript-view.png)

## Cleanup

When you're done, remove all deployed resources.

### Mac / Linux / CloudShell

```bash
cd deployment-package
./cleanup.sh
```

### Windows

```powershell
cd deployment-package
.\cleanup.ps1
```

Both scripts will:
1. Show all resources that will be deleted
2. Ask you to type `yes-delete-all` to confirm
3. Empty the S3 bucket
4. Delete the entire CloudFormation stack (all ~19 resources)
5. Wait for deletion to complete (10-15 min due to CloudFront)

After cleanup, the scripts show optional commands to delete Lambda CloudWatch Log Groups if you want a completely clean slate.

### Manual Cleanup

If you prefer to clean up manually:

```bash
# Get bucket name
BUCKET=$(aws cloudformation describe-stacks \
  --stack-name YOUR_STACK_NAME \
  --region YOUR_REGION \
  --query 'Stacks[0].Outputs[?OutputKey==`S3BucketName`].OutputValue' \
  --output text)

# Empty bucket
aws s3 rm s3://$BUCKET --recursive --region YOUR_REGION

# Delete stack
aws cloudformation delete-stack --stack-name YOUR_STACK_NAME --region YOUR_REGION

# Wait for deletion
aws cloudformation wait stack-delete-complete --stack-name YOUR_STACK_NAME --region YOUR_REGION
```

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| "Parameter ProjectName failed to satisfy constraint" | Uppercase or special characters in name | Use only lowercase letters, numbers, and hyphens |
| "Failed to load configuration" | CloudFront still propagating | Wait 2-3 minutes, then hard refresh (Ctrl+Shift+R) |
| No sessions found | Wrong log group or no conversations yet | Verify log group name, try increasing time range to 72h |
| Browse Logs returns 0 sessions | Log group doesn't contain AI Agent logs | Check your Connect instance has CloudWatch logging enabled |
| API errors when fetching spans | Wrong Assistant ID | Verify the UUID in Connect console |
| Stack creation fails | Missing IAM permissions | Ensure your user can create S3, CloudFront, Lambda, API Gateway, IAM roles |

## Security

- S3 bucket is private — no public access, all public access blocked
- CloudFront uses Origin Access Control (OAC) to access S3
- API Gateway has CORS configured for browser access
- Lambda functions use least-privilege IAM roles
- All traffic is HTTPS

## Package Contents

```
deployment-package/
├── cloudformation-template.yaml   # Full CloudFormation template
├── v2_frontend.html               # Frontend application
├── deploy.sh                      # Mac/Linux/CloudShell deploy script
├── deploy.ps1                     # Windows PowerShell deploy script
├── cleanup.sh                     # Mac/Linux/CloudShell cleanup script
├── cleanup.ps1                    # Windows PowerShell cleanup script
├── QUICKSTART.md                  # 5-minute quick start
├── DEPLOYMENT_GUIDE.md            # Detailed deployment guide
├── README.md                      # Package readme
└── README-WINDOWS.md              # Windows-specific instructions
```

## Screenshots



## Version

v2.0 — Production Ready

## Author

Mo Miah (mmiahaws)


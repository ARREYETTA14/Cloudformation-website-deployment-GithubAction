# Deploying a Static Website to AWS S3 Using CloudFormation and GitHub Actions CI/CD Pipeline

This project sets up a fully automated CI/CD pipeline that deploys a production-ready public static website to AWS S3 using **AWS CloudFormation** for infrastructure provisioning and **GitHub Actions** for automated deployment. Authentication between GitHub and AWS is handled securely using **IAM User Access Keys** stored as encrypted GitHub Secrets.

Every time you push code to the `main` branch, GitHub Actions automatically provisions your infrastructure, applies your bucket policy, uploads your website files, and prints your live URL — all without any manual intervention.

> 💡 **Substitution Guide:** Anything written in `<angle-brackets>` throughout this README is something **you must replace** with your own values before running any command or saving any file.

---

## ⚠️ Important Architecture Note

The `BucketPolicy` (public read access) is intentionally **excluded** from `template.yaml`. AWS enforces an account-level `AWS::EarlyValidation::PropertyValidation` hook that blocks CloudFormation from deploying any bucket policy with `Principal: '*'`. As a result, the public read policy is applied as a dedicated step inside the GitHub Actions workflow using the AWS CLI after the stack is created.

---

## 📂 Project Directory Structure

Your project folder must match this structure exactly before pushing to GitHub:

```text
<your-repo-name>/
├── .github/
│   └── workflows/
│       └── deploy.yml        # GitHub Actions CI/CD pipeline workflow
├── template.yaml             # CloudFormation Infrastructure-as-Code Blueprint
├── index.html                # Main Landing Page
├── error.html                # Custom 404 Error Page
└── styles.css                # Stylesheet (if applicable)
```

---

## 📝 Files Overview

### 1. `template.yaml` — CloudFormation Blueprint
Provisions the S3 bucket with public access block settings disabled and static website hosting enabled. The bucket policy is excluded due to AWS account-level validation restrictions and is applied separately in the pipeline.

### 2. `deploy.yml` — GitHub Actions Workflow
The CI/CD pipeline that runs automatically on every push to `main`. It authenticates with AWS using IAM Access Keys stored as GitHub Secrets, deploys the CloudFormation stack, applies the public bucket policy, syncs website files, and prints the live URL.

---

## 🚀 Complete Step-by-Step Setup Guide

---

### PART 1: AWS Console Setup

---

#### Step 1: Create the IAM User for GitHub Actions

This is the AWS identity that GitHub Actions will use to authenticate and deploy resources.

1. Open the [AWS IAM Console](https://console.aws.amazon.com/iam)
2. In the left sidebar click **Users** → click **Create user**
3. Enter the **User name**:
   ```
   <your-iam-username>
   ```
   > 🚨 SUBSTITUTE: Choose a name e.g. `github-actions-user`
4. Click **Next**
5. Select **Attach policies directly**
6. In the permissions search box, find and attach these two policies:
   - ✅ `AmazonS3FullAccess`
   - ✅ `AWSCloudFormationFullAccess`
7. Click **Next** → click **Create user**

> ✅ Your IAM user is now created with the permissions needed to deploy your website.

---

#### Step 2: Generate Access Keys for the IAM User

These keys are what GitHub Actions will use to authenticate with AWS.

1. Click on the newly created **`<your-iam-username>`** from the Users list
2. Navigate to the **Security credentials** tab
3. Scroll down to the **Access keys** section and click **Create access key**
4. For **Use case** select **Third-party service** → click **Next** → click **Create access key**
5. ⚠️ **CRITICAL**: Copy both values immediately — the Secret access key is only shown once:
   - **Access key ID** — starts with `AKIA...`
   - **Secret access key** — a long alphanumeric string
6. Click **Done**

> ✅ Keep these two values safe — you will paste them into GitHub Secrets in the next step.

---

### PART 2: GitHub Repository Setup

---

#### Step 3: Create Your GitHub Repository

1. Go to [github.com](https://github.com) and sign in
2. Click the **+** icon top right → **New repository**
3. Fill in the details:
   - **Repository name**: `<your-repo-name>` 🚨 SUBSTITUTE with your chosen repo name
   - **Visibility**: Public or Private (both work)
   - ⚠️ **CRITICAL**: Do **NOT** check any initialization boxes (no README, no .gitignore, no license) — leave it completely empty
4. Click **Create repository**
5. Copy the **HTTPS clone URL** shown on the next page

---

#### Step 4: Add AWS Credentials as GitHub Secrets

This is where you securely store your IAM Access Keys so the workflow can use them without hardcoding them anywhere.

1. Go to your GitHub repository page
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret** and add these two secrets exactly:

| Secret Name | Value |
|---|---|
| `AWS_ACCESS_KEY_ID` | Paste your **Access key ID** from Step 2 |
| `AWS_SECRET_ACCESS_KEY` | Paste your **Secret access key** from Step 2 |

> ✅ GitHub encrypts these secrets — they are never exposed in logs or to other users.

---

#### Step 5: Clone the Repository Locally

Open your terminal in VS Code (**Ctrl + `**) and run:

> 🚨 SUBSTITUTE: Replace `<your-https-clone-url>` with the URL you copied from GitHub
> 🚨 SUBSTITUTE: Replace `<your-repo-name>` with your repository name

```bash
git clone <your-https-clone-url>
cd <your-repo-name>
```

---

### PART 3: Configure Your Project Files

---

#### Step 6: Update `template.yaml`

Open `template.yaml` and substitute the default bucket name:

> 🚨 SUBSTITUTE: Replace `your-unique-bucket-name` with your globally unique S3 bucket name
> - Must be lowercase
> - No spaces or special characters except hyphens
> - Must be globally unique across all AWS accounts
> - Example: `my-portfolio-site-123456789012`

The relevant line to change:
```yaml
Default: 'your-unique-bucket-name'
```

---

#### Step 7: Set Up the GitHub Actions Workflow File

Create the folder structure and workflow file:

1. Inside your project root, create a folder named `.github`
2. Inside `.github`, create a folder named `workflows`
3. Inside `workflows`, place the `deploy.yml` file from this project

Then open `deploy.yml` and substitute **all** the following values:

| Placeholder | What to replace it with |
|---|---|
| `<your-aws-region>` | Your AWS region e.g. `us-east-1`, `sa-east-1` |
| `<your-stack-name>` | Your preferred CloudFormation stack name e.g. `MyWebsiteStack` |
| `<your-bucket-name>` | Your unique S3 bucket name (same as in `template.yaml`) |

> ⚠️ `<your-bucket-name>` appears **3 times** in `deploy.yml` — make sure you replace all 3 occurrences.

> ✅ No role ARN or account ID needed — the workflow authenticates using `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` directly from your GitHub Secrets.

---

### PART 4: Deploy

---

#### Step 8: Push to GitHub to Trigger the Pipeline

Once all files are in place and all substitutions are done, push everything to GitHub:

```bash
# Stage all your project files
git add .

# Commit with a descriptive message
git commit -m "feat: deploy static website via CloudFormation and GitHub Actions"

# Push to the main branch to trigger the pipeline
git push origin main
```

> ✅ This push will automatically trigger your GitHub Actions workflow.

---

#### Step 9: Monitor the Pipeline

1. Go to your repository on **GitHub.com**
2. Click the **Actions** tab at the top
3. Click on the running workflow to watch it live
4. You will see these steps execute in order:
   - ✅ Checkout Code
   - ✅ Configure AWS Credentials
   - ✅ Verify AWS Identity
   - ✅ Deploy CloudFormation Stack
   - ✅ Apply Public Bucket Policy
   - ✅ Sync Site Assets to S3
   - ✅ Print Live Website URL

5. Once complete, expand the **Print Live Website URL** step to find your live URL:
   ```
   http://<your-bucket-name>.s3-website-<your-aws-region>.amazonaws.com
   ```

6. Copy the URL, paste it in your browser and your website is live! 🎉

---

## 🛑 Infrastructure Clean-Up Sequence

When you want to tear down all AWS resources and stop incurring costs:

> 🚨 SUBSTITUTE: Replace `<your-bucket-name>`, `<your-region>` and `<your-stack-name>` with your actual values

```bash
# 1. Empty the S3 bucket (must be empty before it can be deleted)
aws s3 rm s3://<your-bucket-name> --recursive

# 2. Delete the S3 bucket
aws s3api delete-bucket \
  --bucket <your-bucket-name> \
  --region <your-region>

# 3. Delete the CloudFormation stack
aws cloudformation delete-stack --stack-name <your-stack-name>
```

> ✅ You can also delete the IAM user and its access keys from the AWS IAM Console if you no longer need them.

---

## 🔐 Security Highlights

- **No hardcoded credentials** — Access Keys are stored as encrypted GitHub Secrets, never in code
- **Least privilege** — the IAM user has only `AmazonS3FullAccess` and `AWSCloudFormationFullAccess`
- **Public access** is controlled at the bucket policy level, not via legacy ACLs
- **`--delete` flag** on S3 sync ensures stale files are removed automatically on each deploy
- **Rotate your keys** periodically in IAM and update the GitHub Secrets to maintain security

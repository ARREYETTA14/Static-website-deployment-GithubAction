# Static-website-deployment-GithubAction
Automated Deployment of a Simple HTML using Github Action 

##  STEP 1: Create and Configure the S3 Bucket
### 1.1 Create the S3 Bucket
- Go to AWS S3 Console
- Click **"Create bucket"**
- Fill in:
    - **Bucket name**: ``my-static-site-bucket`` (must be unique globally)
    - **Region**: Choose your region (*e.g., ``us-east-1``*)
- Under **Block Public Access**, uncheck **“Block all public access”**
- Confirm the warning checkbox acknowledging the bucket will become public
- Click **Create bucket**

### 🌐 1.2 Enable Static Website Hosting
- Click your newly created bucket name
- Go to the **Properties** tab
- Scroll down to **Static website hosting** and click **Edit**.
- Select **Enable**
- **Index document**: ``index.html``
- **Error document**: ``(optional) error.html``
- Click **Save changes**

### 🔐 1.3 Make Your Bucket Public (Set Bucket Policy)
- Go to the **Permissions** tab
- Scroll to **Bucket policy** and click **Edit**
- Paste the following JSON configuration, ensuring you replace ``my-static-site-bucket`` with your actual bucket name:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-static-site-bucket/*"
    }
  ]
}

```
- Click **Save changes**

## STEP 2: Prepare Your GitHub Repository
### 🔐 2.1 Setting Up AWS Credentials in GitHub
Before your workflow can deploy to AWS, GitHub needs permission to access your AWS account. This is done securely through GitHub Secrets.

### Step-by-step: Add AWS credentials to GitHub
- Go to your repository on GitHub.
- Click on the **Settings** tab
- In the left sidebar, expand **Secrets and variables** → Click **Actions**.
- Add the following two secrets using the credentials generated from an AWS IAM user equipped with ``AmazonS3FullAccess``:
```txt
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

## 🗂️ 3. Project Structure

Ensure your project matches this exact file structure:
```bash
my-site/
├── index.html
└── .github/
    └── workflows/
        └── deploy.yml
```
- Create a repo on GitHub (e.g., my-site)
- Add ``index.html`` with your web content (e.g.,):
```html
<h1>Welcome to My GitHub Actions S3 Website!</h1>
```
- Populate ``.github/workflows/deploy.yml`` with the updated configuration below:
```yaml
name: Deploy to S3

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v7

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v6
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to S3
        run: aws s3 sync . s3://my-static-site-bucket --delete --exclude ".git*"

```

Swap out ``my-static-site-bucket`` and ``aws-region`` with your actual values.

- Commit and Push everything to the ``main`` branch.

**🎉 GitHub Actions will now automatically deploy your site to S3 whenever you push new code. **

- You can view its progress in your repository's **Actions** tab and access your website through the bucket website endpoint at:
``http://my-static-site-bucket.s3-website-us-east-1.amazonaws.com``


# Using OpenID Connect (OIDC)

Switching to **OpenID Connect (OIDC)** is an industry best practice. Instead of saving permanent AWS Access Keys inside your GitHub repository (which can easily leak if misconfigured), GitHub Actions dynamically requests **temporary**, **short-lived credentials** directly from AWS that expire automatically after one hour.

## STEP 1: Create the OIDC Identity Provider in AWS
1. Log into your **AWS Management Console**.
2. Navigate to the **IAM Console** (Identity and Access Management).
3. In the left sidebar, click **Identity providers** under *Access management*.
4. Click **Add provider**.
5. Configure these exact settings:
	• **Provider type**: Select **OpenID Connect**.
	• **Provider URL**: Paste ``token.actions.githubusercontent.com``
	• Audience: Type ``sts.amazonaws.com``
6. Click Add provider.

## STEP 2: Create a Secure IAM Role for GitHub Actions

Now, you need to create an AWS role that your GitHub pipeline is allowed to assume.

1. In the left sidebar of the **IAM Console**, click **Roles** and then click **Create role**.
2. Select Custom trust policy under *Trusted entity type*
3. Paste the following JSON block into the policy editor. 🚨 CRITICAL: Replace ``<YOUR_AWS_ACCOUNT_ID>``, ``<YOUR_GITHUB_ORGANIZATION_OR_USER>``, and ``<YOUR_GITHUB_REPO_NAME>`` with your actual deployment details:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<YOUR_AWS_ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringLike": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
          "token.actions.githubusercontent.com:sub": "repo:<YOUR_GITHUB_USERNAME_OR_ORG>/<YOUR_REPOSITORY_NAME>:*"
        }
      }
    }
  ]
}
```
4. Click **Next**
5. On the *Add permissions screen*, search for and check the box next to ``AmazonS3FullAccess`` (or attach a limited policy that only allows writes to your specific bucket).
6. Click **Next**.
7. **Role name**: Enter ``github-s3-deploy-role``.
8. Review your choices and click **Create role**.
9. Copy the **ARN** string of your new role (it will look like arn:aws:iam::123456789012:role/github-s3-deploy-role).

## STEP 3: Update Your GitHub Workflow File
Before you configure your workflow, you need to make the Role ARN available to it. You'll store it as a repository variable in GitHub, not a secret, because the ARN itself isn't sensitive data.

- First, open your GitHub repository and click **Settings**.
- In the left sidebar, scroll down to **Secrets and variables**, then click **Actions**.
- Then click the **Variables** tab (not Secrets). Click **New repository variable** – you can put the name as **AWS_GITHUB_ROLE**.
- Set the Value to your **Role ARN**
- Click **Add variable**

With AWS and GitHub fully configured, you now need to update your workflow to request an OIDC token and use it to authenticate.

- Your workflow must declare ``id-token: write``. Without this, GitHub won't issue an OIDC token to the runner.
For example:
```yaml
name: Deploy to AWS S3
 
on:
  push:
    branches:
      - main
 
permissions:
  id-token: write
  contents: read
 
jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
 
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
 
      - name: Configure AWS credentials via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ vars.AWS_ROLE_ARN }}
          aws-region: us-east-2
 
      - name: Verify AWS identity
        run: aws sts get-caller-identity
 
      - name: Deploy to S3
        run: |
          aws s3 sync ./code s3://your-bucket-name
```

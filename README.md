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



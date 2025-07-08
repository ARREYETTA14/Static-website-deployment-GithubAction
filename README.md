# Static-website-deployment-GithubAction
Automated Deployment of a Simple HTML using Github Action 

## 🔐 1. Setting Up AWS Credentials in GitHub
Before your workflow can deploy to AWS, GitHub needs permission to access your AWS account. This is done securely through GitHub Secrets.

### 1.1 Step-by-step: Add AWS credentials to GitHub
- Go to your GitHub repo
- Click on the **Settings** tab
- In the left sidebar, scroll to:
```nginx
Secrets and variables → Actions
```
- Click the **“New repository secret”** button
- Add two secrets:
```txt
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```
Using the Principle of **Least Privilege**, to deploy to S3, the IAM user should have the ``AmazonS3FullAccess`` policy.

## 🗂️ 2. Project Structure
```bash
my-site/
├── index.html
└── .github/
    └── workflows/
        └── deploy.yml
```
- Create a repo on GitHub (e.g., my-site)
- Add ``index.html`` with content like:
```html
<h1>Welcome to My GitHub Actions S3 Website!</h1>
```
- Create the folder ``.github/workflows`` and add the ``deploy.yml``.
- Add AWS secrets as shown above.
- Push everything to the ``main`` branch.
**🎉 GitHub Actions will now automatically deploy your site to S3 whenever you push new code. **

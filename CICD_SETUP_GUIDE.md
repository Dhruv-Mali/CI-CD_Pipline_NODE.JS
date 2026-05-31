# CI/CD Pipeline Setup Guide

This project uses GitHub Actions for automated testing, building, and deployment to AWS.

## 📋 Prerequisites

- GitHub repository with the code
- AWS Account with Elastic Beanstalk configured
- Node.js 18+ installed locally

## 🔑 Step 1: Configure GitHub Secrets

Go to your GitHub repository → **Settings → Secrets and variables → Actions** and add the following secrets:

### AWS Credentials
```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION (e.g., us-east-1)
```

### AWS Elastic Beanstalk Configuration
```
AWS_S3_BUCKET (bucket name for deployment packages)
AWS_EB_APP_NAME (your Elastic Beanstalk application name)
AWS_EB_ENV_NAME (your Elastic Beanstalk environment name)
```

### Application Configuration
```
VITE_API_URL (Frontend API URL, e.g., https://api.yourdomain.com)
```

## 📝 Step 2: Create AWS IAM User

1. Go to **AWS Console → IAM → Users → Create User**
2. Give it a name: `cicd-deployment-user`
3. Attach policies:
   - `AWSElasticBeanstalkFullAccess`
   - `AmazonS3FullAccess`
   - `IAMReadOnlyAccess`

4. Generate Access Key ID and Secret Access Key
5. Store these securely and add them to GitHub Secrets

## 🚀 Step 3: Set Up Elastic Beanstalk

### Option A: Using AWS Console (Recommended for beginners)

1. **Create Application:**
   - AWS Console → Elastic Beanstalk → Create Application
   - Application name: `your-app-name`

2. **Create Environment:**
   - Environment name: `production` (or `staging`)
   - Platform: Node.js 18.x
   - Application code: Sample application (will be replaced by CI/CD)

3. **Configure Environment Variables:**
   - Go to Configuration → Software
   - Add environment properties:
     ```
     NODE_ENV=production
     PORT=8081
     MONGODB_URI=<your-mongodb-uri>
     JWT_SECRET=<your-jwt-secret>
     FIREBASE_API_KEY=<firebase-config>
     STRIPE_SECRET_KEY=<your-stripe-key>
     SMTP_USER=<email>
     SMTP_PASS=<email-password>
     ```

### Option B: Using AWS CLI

```bash
# Create application
aws elasticbeanstalk create-application \
  --application-name your-app-name \
  --region us-east-1

# Create environment
aws elasticbeanstalk create-environment \
  --application-name your-app-name \
  --environment-name production \
  --cname-prefix your-app-prod \
  --solution-stack-name "64bit Amazon Linux 2023 v6.0.1 running Node.js 18" \
  --region us-east-1
```

## 📁 Step 4: Project Structure for AWS Deployment

Ensure your project has a proper `.ebextensions` folder for Elastic Beanstalk configuration:

```bash
mkdir -p .ebextensions
```

See `.ebextensions/` in this repository for sample configurations.

## 🔄 Step 5: Environment Configuration

### Backend (.env)
Create `.env` file in root (not committed to git):
```
MONGODB_URI=mongodb+srv://user:password@cluster.mongodb.net/dbname
JWT_SECRET=your-secret-key
FIREBASE_API_KEY=your-firebase-key
STRIPE_SECRET_KEY=sk_test_...
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password
NODE_ENV=production
PORT=8081
```

### Frontend (.env.local)
Create `client/.env.local` (not committed to git):
```
VITE_API_URL=https://your-api-domain.com
```

## 📊 Pipeline Workflow

```
Push to Main/Develop
    ↓
Lint & Code Quality (ESLint)
    ↓
Build Frontend (Vite)
Build Backend (Node.js)
    ↓
Deploy to AWS (Main branch only)
    ↓
Notify Status
```

## ✅ Running the Pipeline

### Automatically (on push):
```bash
git push origin main
```
Pipeline runs automatically on GitHub Actions.

### View Results:
1. Go to your GitHub repository
2. Click **Actions** tab
3. Select the workflow run
4. Monitor real-time progress

## 🐛 Troubleshooting

### Lint Failures
```bash
# Fix ESLint errors locally
npm run lint --prefix client -- --fix
```

### Build Failures
```bash
# Test build locally
npm run build --prefix client
npm install
```

### AWS Deployment Issues

1. **Invalid AWS Credentials:**
   - Verify Access Key ID and Secret Access Key in GitHub Secrets
   - Ensure IAM user has correct permissions

2. **Elastic Beanstalk Environment Not Found:**
   - Check `AWS_EB_APP_NAME` and `AWS_EB_ENV_NAME` spelling
   - Verify the environment exists in AWS Console

3. **S3 Bucket Error:**
   - Verify bucket exists and is in same AWS region
   - Ensure IAM user has S3 full access

4. **Deployment Timeout:**
   - Check Elastic Beanstalk logs in AWS Console
   - Verify environment has enough resources
   - Check application logs for runtime errors

### View Elastic Beanstalk Logs
```bash
aws elasticbeanstalk retrieve-environment-info \
  --application-name your-app-name \
  --environment-name production \
  --info-type tail \
  --region us-east-1
```

## 🔐 Security Best Practices

1. ✅ Never commit `.env` files
2. ✅ Use GitHub Secrets for sensitive data
3. ✅ Rotate AWS credentials regularly
4. ✅ Use IAM roles with minimal required permissions
5. ✅ Enable branch protection rules on main
6. ✅ Require PR reviews before merge

## 📈 Monitoring & Scaling

### CloudWatch Monitoring:
- AWS Console → CloudWatch → Dashboards
- Monitor CPU, memory, network metrics
- Set up alarms for threshold breaches

### Elastic Beanstalk Health:
- View environment health dashboard
- Check recent deployments
- Review application logs

## 🛠️ Local Development

To test build locally:
```bash
npm install
npm run build
npm run lint --prefix client
```

## 📚 Useful Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [AWS Elastic Beanstalk Docs](https://docs.aws.amazon.com/elasticbeanstalk/)
- [Node.js Best Practices](https://nodejs.org/en/docs/)
- [GitHub Secrets Management](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)

---

**Next Steps:** 
1. Add GitHub Secrets
2. Set up Elastic Beanstalk environment
3. Configure environment variables
4. Push code and monitor first deployment

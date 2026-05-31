# GitHub Actions CI/CD Pipeline - Quick Reference

## 📊 Pipeline Stages

### 1. Lint & Code Quality
- Runs ESLint on client code
- Checks for code quality issues
- **Fails on:** Linting errors (as per .eslintrc)
- **Runs:** On every push and PR

### 2. Build Frontend
- Installs client dependencies
- Builds React + Vite production bundle
- Uploads artifacts for deployment
- **Runs:** After lint passes, on every push/PR

### 3. Build Backend  
- Installs root dependencies
- Verifies Node.js syntax
- **Runs:** After lint passes, on every push/PR

### 4. Deploy to AWS
- Downloads frontend build artifacts
- Packages application for deployment
- Uploads to S3
- Creates new Elastic Beanstalk version
- Updates environment
- **Runs:** Only on main branch when push succeeds
- **Requires:** All previous jobs to pass

## 🔐 Required GitHub Secrets

Create these in Repository → Settings → Secrets and variables → Actions:

```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY  
AWS_REGION
AWS_S3_BUCKET
AWS_EB_APP_NAME
AWS_EB_ENV_NAME
VITE_API_URL
```

## 🚀 How to Trigger

### Automatic Triggers
```bash
# Trigger CI (lint + build only)
git push origin develop

# Trigger CI + Deploy (lint + build + deploy)
git push origin main
```

### Manual Trigger (GitHub CLI)
```bash
# Create a deployment run
gh workflow run ci-cd.yml --ref main
```

## 📈 Monitoring

1. **GitHub Actions Tab:** `github.com/yourrepo/actions`
2. **Real-time logs:** Click the running workflow
3. **Deployment status:** Check AWS Elastic Beanstalk dashboard
4. **Application logs:** CloudWatch or Elastic Beanstalk console

## ✅ Success Indicators

- ✅ All workflow jobs turn green
- ✅ Lint passes without errors
- ✅ Frontend and backend build successfully
- ✅ Elastic Beanstalk shows "Ready" status
- ✅ Application is accessible at your domain

## ❌ Common Failures

| Issue | Solution |
|-------|----------|
| Lint fails | Run `npm run lint --prefix client -- --fix` locally |
| Build fails | Run `npm run build --prefix client` locally |
| AWS auth fails | Check AWS credentials in GitHub Secrets |
| Deployment timeout | Check Elastic Beanstalk logs in AWS Console |
| Frontend not loading | Verify VITE_API_URL environment variable |

## 🔄 Workflow File Location

- Main workflow: [`.github/workflows/ci-cd.yml`](.github/workflows/ci-cd.yml)
- Edit workflow: GitHub → Settings → Actions → Edit workflow

## 📝 Logs & Debugging

### View Workflow Logs
1. GitHub → Actions → Select workflow run
2. Click job name to expand
3. View real-time or complete logs

### View Elastic Beanstalk Deployment Logs
```bash
# SSH into EB instance
aws ec2-instance-connect open-session --target <instance-id>

# View app logs
tail -f /var/log/nodejs/nodejs.log
```

### View CloudWatch Logs
```bash
aws logs tail /aws/elasticbeanstalk/your-app-name/production --follow
```

## 🛠️ Local Testing

Before pushing, test locally:

```bash
# Install all dependencies
npm install && npm install --prefix client

# Lint client code
npm run lint --prefix client

# Build frontend
npm run build --prefix client

# Test backend start
npm start
```

## 📅 Pipeline Timing

- **Lint:** ~2 minutes
- **Frontend Build:** ~3 minutes  
- **Backend Build:** ~1 minute
- **AWS Deployment:** ~5-10 minutes
- **Total:** ~15 minutes

## 🔗 Useful Links

- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [AWS Elastic Beanstalk Docs](https://docs.aws.amazon.com/elasticbeanstalk/)
- [Workflow Status Badge](#badge)

---

**For detailed setup instructions, see [CICD_SETUP_GUIDE.md](CICD_SETUP_GUIDE.md)**
**For deployment configuration, see [DEPLOYMENT_CONFIG.md](DEPLOYMENT_CONFIG.md)**

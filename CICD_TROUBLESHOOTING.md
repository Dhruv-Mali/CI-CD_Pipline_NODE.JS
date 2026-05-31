# CI/CD Pipeline Troubleshooting Guide

## Common Issues and Solutions

### 1. Lint Failures

#### Issue: "ESLint errors prevent build"

**Symptoms:**
- Red X on workflow run
- Message: "eslint: errors found"

**Solutions:**

```bash
# View all linting issues
npm run lint --prefix client

# Auto-fix fixable issues
npm run lint --prefix client -- --fix

# Fix specific file
npx eslint client/src/components/SomeComponent.jsx --fix
```

**Common lint errors:**
- Unused imports: Remove unused variables/imports
- Missing dependencies in useEffect: Add to dependency array
- Undefined variables: Check variable declaration

---

### 2. Build Failures

#### Issue: "Frontend build fails"

**Symptoms:**
- Build job turns red
- Error mentions Vite or webpack

**Solutions:**

```bash
# Install dependencies fresh
rm -rf client/node_modules client/package-lock.json
npm install --prefix client

# Test build locally
npm run build --prefix client

# Check for import errors
npm run dev --prefix client  # Run dev server to see errors
```

**Common build errors:**
- Missing environment variables: Check `.env.local` or GitHub Secrets
- Module not found: Verify import paths are correct
- TypeScript errors: Check `tsconfig.json` configuration

---

### 3. AWS Deployment Failures

#### Issue: "Deployment to AWS fails"

**Symptoms:**
- Deploy job shows red
- Error: "Invalid AWS credentials"

**Solutions:**

**A. Check AWS Credentials:**

1. Go to GitHub repo → Settings → Secrets and variables → Actions
2. Verify these secrets exist:
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`
   - `AWS_REGION`

3. Verify IAM user permissions:
   ```bash
   aws iam list-attached-user-policies --user-name cicd-deployment-user
   ```

4. Regenerate credentials:
   - AWS Console → IAM → Users
   - Select user → Security credentials
   - Create new access key
   - Update GitHub Secrets

**B. Check Elastic Beanstalk Configuration:**

```bash
# Verify EB app exists
aws elasticbeanstalk describe-applications \
  --region us-east-1

# Verify EB environment exists
aws elasticbeanstalk describe-environments \
  --application-name your-app-name \
  --region us-east-1
```

If not found, create them:
```bash
# Create application
aws elasticbeanstalk create-application \
  --application-name your-app-name

# Create environment
aws elasticbeanstalk create-environment \
  --application-name your-app-name \
  --environment-name production \
  --solution-stack-name "64bit Amazon Linux 2023 v6.0.1 running Node.js 18"
```

#### Issue: "S3 bucket not found"

**Solutions:**

```bash
# List all S3 buckets
aws s3 ls

# Create S3 bucket if missing
aws s3 mb s3://my-deployment-bucket --region us-east-1

# Verify bucket is in correct region
aws s3api get-bucket-location --bucket my-deployment-bucket
```

#### Issue: "Deployment times out"

**Symptoms:**
- Deployment waits forever
- No error message

**Solutions:**

1. **Check Elastic Beanstalk logs:**
   ```bash
   aws elasticbeanstalk retrieve-environment-info \
     --application-name your-app-name \
     --environment-name production \
     --info-type tail \
     --region us-east-1
   ```

2. **SSH into instance:**
   ```bash
   # Find instance ID
   aws ec2 describe-instances \
     --filters "Name=tag:elasticbeanstalk:environment-name,Values=production"
   
   # Connect to instance
   aws ec2-instance-connect open-session --target <instance-id>
   
   # Check logs
   tail -f /var/log/nodejs/nodejs.log
   ```

3. **Common causes:**
   - Missing environment variables
   - Database connection issues
   - Insufficient instance resources
   - Node.js startup errors

---

### 4. Environment Variable Issues

#### Issue: "Application crashes: Cannot find environment variable"

**Symptoms:**
- Deployment succeeds but app won't start
- Error: `ReferenceError: process.env.VARIABLE is not defined`

**Solutions:**

1. **Verify all variables are set in AWS:**
   ```bash
   aws elasticbeanstalk describe-environments \
     --application-name your-app-name \
     --environment-names production \
     --query 'Environments[0].OptionSettings' \
     --region us-east-1
   ```

2. **Add missing variables:**
   ```bash
   aws elasticbeanstalk update-environment \
     --application-name your-app-name \
     --environment-name production \
     --option-settings \
       Namespace=aws:elasticbeanstalk:application:environment,OptionName=MONGODB_URI,Value="your-uri" \
     --region us-east-1
   ```

3. **For GitHub Actions secrets:**
   - Go to repo → Settings → Secrets → Add `VITE_API_URL`

#### Issue: "Frontend can't reach API"

**Symptoms:**
- Frontend loads but API calls fail
- Error: `Failed to fetch from undefined`

**Solutions:**

1. **Check VITE_API_URL:**
   ```bash
   # In GitHub Actions logs, verify it's set
   # Should appear in build logs
   ```

2. **Update frontend code:**
   ```javascript
   // client/src/utils/api.js
   const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:8081';
   ```

3. **For AWS deployment:**
   - Add GitHub Secret: `VITE_API_URL=https://api.yourdomain.com`

---

### 5. Port and Network Issues

#### Issue: "Application can't start: Port already in use"

**Symptoms:**
- Error: `EADDRINUSE: address already in use :::8081`

**Solutions:**

```bash
# Check if port is in use
lsof -i :8081

# Kill process using port
kill -9 <PID>

# Or use different port in .env
PORT=8082
```

#### Issue: "Cannot connect to MongoDB"

**Symptoms:**
- Application starts but can't save data
- Error: `MongoError: connect ECONNREFUSED`

**Solutions:**

```bash
# Test MongoDB connection
mongosh "mongodb+srv://user:pass@cluster.mongodb.net/dbname"

# If using MongoDB Atlas:
1. Verify IP is whitelisted: MongoDB Atlas → Security → Network Access
2. Check username/password in connection string
3. Ensure @cluster in URI matches your cluster name

# Check connection string format
mongodb+srv://username:password@cluster-name.mongodb.net/database-name
                ^^^^^^^^ ^^^^^^^^                  ^^^^^^^^^^^^^
              user creds                          cluster name
```

---

### 6. GitHub Actions Specific Issues

#### Issue: "Workflow file syntax error"

**Symptoms:**
- Workflow won't run
- Error: `Invalid workflow file`

**Solutions:**

1. **Validate YAML syntax:**
   - Check `.github/workflows/ci-cd.yml`
   - Look for indentation errors
   - Use [YAML validator](https://www.yamllint.com/)

2. **Common issues:**
   - Tabs instead of spaces (use 2 spaces)
   - Missing colons after keys
   - Incorrect boolean values (use `true`/`false`, not `True`/`False`)

#### Issue: "Artifact not found"

**Symptoms:**
- Build job passes but deployment fails
- Error: `Could not find artifact`

**Solutions:**

1. **Verify build output exists:**
   ```bash
   npm run build --prefix client
   ls -la client/dist/
   ```

2. **Check artifact path in workflow:**
   - Ensure upload path matches download path
   - Current: `client/dist/`

---

### 7. Performance Issues

#### Issue: "Deployment is too slow"

**Solutions:**

1. **Cache npm dependencies:**
   - Workflow already uses `cache: 'npm'`
   - Verify cache-action is present

2. **Parallel jobs:**
   - Lint, frontend build, and backend build run in parallel
   - Only deploy runs sequentially

3. **Optimize build:**
   ```bash
   # Check build size
   npm run build --prefix client
   du -sh client/dist/
   
   # Optimize if needed
   # - Remove unused dependencies
   # - Enable code splitting
   # - Use tree-shaking
   ```

---

### 8. Security Issues

#### Issue: "Secrets exposed in logs"

**Symptoms:**
- GitHub Actions logs show API keys or tokens

**Solutions:**

1. **Never print secrets:**
   ```yaml
   # ❌ BAD
   - run: echo ${{ secrets.AWS_SECRET_ACCESS_KEY }}
   
   # ✅ GOOD
   - name: Deploy
     env:
       AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
   ```

2. **Rotate exposed secrets immediately**

---

## Debugging Steps Checklist

When something fails:

- [ ] Check workflow logs on GitHub
- [ ] Read error messages carefully
- [ ] Test commands locally first
- [ ] Verify all GitHub Secrets are set
- [ ] Verify AWS resources exist and are configured
- [ ] Check environment variables in AWS EB
- [ ] Review recent code changes
- [ ] Check AWS CloudWatch logs
- [ ] Verify database connectivity
- [ ] Test payment/email services separately

---

## Getting Help

**Resources:**
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [AWS Elastic Beanstalk Docs](https://docs.aws.amazon.com/elasticbeanstalk/)
- [Node.js Error Handling](https://nodejs.org/en/docs/guides/nodejs-error-handling/)
- [MongoDB Troubleshooting](https://docs.mongodb.com/manual/reference/method/db.serverStatus/)

**If stuck:**
1. Check CloudWatch logs: AWS Console → CloudWatch → Logs
2. SSH into instance and check application logs
3. Run workflow with `debug: true` for verbose output
4. Create GitHub Issue with workflow logs

---

**Last Updated:** May 31, 2026

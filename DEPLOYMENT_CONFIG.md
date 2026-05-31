# Deployment Configuration Guide

## Environment Variables for AWS Deployment

Copy and configure these environment variables in AWS Elastic Beanstalk Console:

### Required Variables

```properties
# Node Environment
NODE_ENV=production
PORT=8081

# Database
MONGODB_URI=mongodb+srv://username:password@cluster-name.mongodb.net/database-name?retryWrites=true&w=majority

# JWT Authentication
JWT_SECRET=your-super-secret-jwt-key-change-this-in-production

# Firebase Configuration
FIREBASE_API_KEY=your-firebase-api-key
FIREBASE_AUTH_DOMAIN=your-app.firebaseapp.com
FIREBASE_PROJECT_ID=your-project-id
FIREBASE_STORAGE_BUCKET=your-project.appspot.com
FIREBASE_MESSAGING_SENDER_ID=your-sender-id
FIREBASE_APP_ID=your-app-id

# Stripe Payment
STRIPE_SECRET_KEY=sk_live_your-stripe-secret-key
STRIPE_PUBLIC_KEY=pk_live_your-stripe-public-key

# Email Configuration
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-specific-password
SMTP_PORT=587
SMTP_HOST=smtp.gmail.com

# Client URL
CLIENT_URL=https://your-domain.com
API_URL=https://api.your-domain.com
```

## Setting Environment Variables

### Method 1: AWS Console (Easiest)

1. Go to **AWS Elastic Beanstalk Console**
2. Select your **Environment**
3. Click **Configuration** → **Software**
4. Scroll to **Environment properties**
5. Add each key-value pair
6. Click **Apply**

### Method 2: AWS CLI

```bash
aws elasticbeanstalk update-environment \
  --application-name your-app-name \
  --environment-name production \
  --option-settings \
    Namespace=aws:elasticbeanstalk:application:environment,OptionName=NODE_ENV,Value=production \
    Namespace=aws:elasticbeanstalk:application:environment,OptionName=MONGODB_URI,Value="mongodb+srv://..." \
  --region us-east-1
```

### Method 3: .env.yaml in .ebextensions

Create `.ebextensions/env.yml`:
```yaml
option_settings:
  aws:elasticbeanstalk:application:environment:
    NODE_ENV: production
    PORT: 8081
    MONGODB_URI: "your-mongodb-uri"
```

## Important Notes

⚠️ **Security Reminders:**
- Never commit `.env` or sensitive configuration to Git
- Use AWS Secrets Manager for highly sensitive data
- Rotate JWT secrets regularly in production
- Use environment-specific Stripe keys (test vs live)

## Database Setup

### MongoDB Atlas Setup:
1. Create cluster at [mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas)
2. Create database user
3. Whitelist IP: Add `0.0.0.0/0` for Elastic Beanstalk (or specific IPs)
4. Copy connection string: `mongodb+srv://user:pass@cluster.mongodb.net/dbname`

## Firebase Setup

1. Go to [Firebase Console](https://console.firebase.google.com)
2. Create project
3. Add web app
4. Copy configuration from project settings
5. Store in environment variables

## Stripe Setup

1. Get API keys from [Stripe Dashboard](https://dashboard.stripe.com)
2. Use test keys for staging, live keys for production
3. Store secret key in environment variables

## SSL/HTTPS Configuration

Elastic Beanstalk uses AWS Certificate Manager (ACM):

1. Go to **AWS Certificate Manager**
2. Request certificate for your domain
3. Complete DNS validation
4. Associate with Elastic Beanstalk environment

## Monitoring & Logging

After deployment, monitor:

```bash
# View recent logs
aws elasticbeanstalk retrieve-environment-info \
  --application-name your-app-name \
  --environment-name production \
  --info-type tail \
  --region us-east-1

# Get environment health
aws elasticbeanstalk describe-environments \
  --application-name your-app-name \
  --environment-names production \
  --region us-east-1
```

## Troubleshooting

### Application fails to start
- Check CloudWatch Logs in AWS Console
- Verify all required environment variables are set
- Test database connection locally

### High memory usage
- Check for memory leaks in application
- Increase instance size in Elastic Beanstalk configuration
- Monitor with CloudWatch

### Slow responses
- Check MongoDB connection pool
- Enable caching for static assets
- Scale up with auto-scaling groups

---

**See CICD_SETUP_GUIDE.md for complete CI/CD setup instructions.**

# AWS Deployment Plan for Farhat Portfolio

## Current Site

This repo is a static portfolio website:

- `index.html`
- `style.css`
- `headshot.png`

There is no build step yet. The site can be previewed locally by opening `index.html` in a browser.

## Goal

Use this small portfolio as a safe AWS practice project before hosting a larger web application. The learning goal is to understand:

- Static file hosting
- CDN delivery
- HTTPS certificates
- Custom domains and DNS
- Deployment updates
- Basic cost and cleanup habits

## Recommended Learning Path

### Phase 1: Fast First Deployment with AWS Amplify Hosting

Use this path first if you want the quickest successful deployment from GitHub.

1. Sign in to AWS.
2. Open AWS Amplify.
3. Choose to host a web app from GitHub.
4. Connect the repo:
   - GitHub repo: `CS22M105/farhat_portfolio`
   - Branch: `master`
5. If Amplify asks for build settings, use:
   - Build command: none
   - Output directory: `/` or `.`
   - Files: `index.html`, `style.css`, `headshot.png`
6. Deploy and test the generated `amplifyapp.com` URL.
7. Make a small change locally, push to GitHub, and confirm Amplify redeploys automatically.

Why this phase matters:

- You get CI/CD quickly.
- You learn GitHub-to-AWS deployment without managing many AWS resources manually.
- It gives you confidence before moving to S3, CloudFront, Route 53, and ACM.

### Phase 2: Real AWS Static Architecture with S3 and CloudFront

Use this path to learn the core AWS services that are useful for larger production apps.

Architecture:

```text
User Browser
    |
    v
CloudFront CDN + HTTPS
    |
    v
Private S3 Bucket
```

Recommended services:

- Amazon S3 for storing the static website files
- Amazon CloudFront for global CDN delivery
- CloudFront Origin Access Control, also called OAC, so S3 is not public
- AWS Certificate Manager, also called ACM, for HTTPS
- Amazon Route 53 for DNS if you use a custom domain

Do not start by making the S3 bucket public. AWS documentation recommends keeping S3 Block Public Access enabled and using CloudFront OAC for secure static websites.

## Phase 2 Console Steps

1. Create an S3 bucket.
   - Example name: `farhat-portfolio-site`
   - Keep Block Public Access enabled.
   - Do not enable public bucket policy access.

2. Upload the website files.
   - Upload `index.html`
   - Upload `style.css`
   - Upload `headshot.png`

3. Create a CloudFront distribution.
   - Origin: the S3 bucket REST endpoint, not the S3 static website endpoint
   - Origin access: Origin Access Control, OAC
   - Viewer protocol policy: Redirect HTTP to HTTPS
   - Default root object: `index.html`
   - Compression: enabled

4. Update the S3 bucket policy.
   - Let CloudFront read objects from the bucket.
   - Restrict access to the CloudFront distribution ARN.

5. Test the CloudFront URL.
   - It will look like `https://dxxxxxxxxxxxxx.cloudfront.net`

6. Optional custom domain.
   - Register or use an existing domain.
   - Create or use a Route 53 hosted zone.
   - Request an ACM certificate in `us-east-1` for CloudFront.
   - Validate the certificate using DNS.
   - Add your domain as a CloudFront alternate domain name.
   - Create Route 53 A and AAAA alias records pointing to CloudFront.

## Phase 2 CLI Setup

The AWS CLI is not currently installed on this machine. Before using CLI deployment commands:

```bash
brew install awscli
aws configure sso
```

If you do not use AWS SSO, use:

```bash
aws configure
```

Then confirm:

```bash
aws sts get-caller-identity
```

## Example CLI Deployment Commands

Set variables:

```bash
BUCKET_NAME=farhat-portfolio-site
DISTRIBUTION_ID=YOUR_CLOUDFRONT_DISTRIBUTION_ID
```

Create a bucket:

```bash
aws s3 mb s3://$BUCKET_NAME --region us-east-1
```

Upload the site:

```bash
aws s3 sync . s3://$BUCKET_NAME \
  --exclude ".git/*" \
  --exclude "AWS_DEPLOYMENT_PLAN.md" \
  --delete
```

Invalidate CloudFront after uploading changes:

```bash
aws cloudfront create-invalidation \
  --distribution-id $DISTRIBUTION_ID \
  --paths "/*"
```

## Example S3 Bucket Policy for CloudFront OAC

Replace:

- `BUCKET_NAME`
- `AWS_ACCOUNT_ID`
- `DISTRIBUTION_ID`

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipalReadOnly",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::BUCKET_NAME/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::AWS_ACCOUNT_ID:distribution/DISTRIBUTION_ID"
        }
      }
    }
  ]
}
```

## Cost Control Checklist

- Create an AWS Budget before experimenting.
- Prefer the smallest possible setup while learning.
- Delete test CloudFront distributions, S3 buckets, Route 53 hosted zones, and ACM certificates when done.
- Remember that Route 53 hosted zones and registered domains can have recurring costs.
- CloudFront invalidations beyond the free monthly allowance can cost money, so avoid unnecessary invalidations while practicing.

## How This Maps to a Bigger Web App Later

For a larger application, this portfolio teaches the frontend hosting layer. A bigger app usually adds:

- Frontend: S3 plus CloudFront, or Amplify Hosting
- Backend API: API Gateway plus Lambda, App Runner, Elastic Beanstalk, or ECS Fargate
- Database: DynamoDB, RDS, or Aurora
- Auth: Amazon Cognito or an external auth provider
- Secrets: AWS Secrets Manager or SSM Parameter Store
- CI/CD: GitHub Actions, AWS CodePipeline, or Amplify branch deploys
- Infrastructure as code: AWS CDK, Terraform, or CloudFormation
- Monitoring: CloudWatch logs, metrics, alarms, and dashboards

Recommended next practice project after this:

1. Deploy this portfolio with Amplify.
2. Deploy the same portfolio with S3 plus CloudFront.
3. Add a contact form using API Gateway, Lambda, and SES.
4. Convert the deployment to infrastructure as code with AWS CDK or Terraform.
5. Use the same pattern for the bigger web app.

## Official AWS References

- AWS Amplify Hosting: https://aws.amazon.com/amplify/hosting/
- Amazon S3 static website hosting: https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html
- Secure static website with CloudFront: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/getting-started-secure-static-website-cloudformation-template.html
- CloudFront OAC for S3 origins: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html
- Route 53 alias to CloudFront: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-to-cloudfront-distribution.html
- ACM DNS validation: https://docs.aws.amazon.com/acm/latest/userguide/dns-validation.html

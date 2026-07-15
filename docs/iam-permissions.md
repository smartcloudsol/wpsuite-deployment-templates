# IAM permissions overview

This document summarizes what to inspect in the WP Suite Deployment Access CloudFormation templates before launch.

The exact IAM statements are defined in the templates under `templates/`. This file is a review guide, not a replacement for reading the generated YAML.

## Root orchestrator

The root template coordinates the four wrapper/wrapped component stack pairs and deployment support custom resources. In Quick Launch, all four pairs can appear in the CloudFormation tree; each wrapped template's `Enabled` parameter controls whether that component creates resources.

Review:

- permissions used by deployment artifact staging
- permissions used by deployment output reporting
- access to deployment-time secrets and configuration
- deployment context passed to wrappers and component parameters resolved from the deployment secret and forwarded to wrapped templates

## WP Suite Cognito

Review:

- Cognito user pool, app client, and optional identity pool resources
- Lambda trigger execution roles
- optional SES and email-template access
- optional Route 53 custom domain alias management
- optional reCAPTCHA secret access

## WP Suite AI Kit Backend

Review:

- API Gateway and Lambda execution roles
- DynamoDB table access
- S3 knowledge-base and upload bucket access
- Bedrock permissions if enabled
- optional WAF and reCAPTCHA settings
- Cognito or token-based admin/frontend authorization settings

## WP Suite Flow Backend

Review:

- API Gateway and Lambda execution roles
- DynamoDB workflow, submission, and template table access
- S3 template and upload bucket access
- SES permissions if email sending is enabled
- optional EventBridge and webhook-related permissions
- Cognito or token-based admin/frontend authorization settings

## WP Suite Static Site Guardian

Review:

- CloudFront distribution and signing resources
- S3 site bucket access
- Lambda@Edge packaging and execution roles
- Cognito-compatible JWT validation configuration
- optional Route 53 and custom domain resources
- optional WP Suite configuration refresh permissions

## Least-privilege expectation

The templates are generated from a CDK-based template factory and should avoid broad `*` permissions where a narrower resource scope is known at template generation time.

If you find a permission that appears broader than necessary, open a Marketplace support request or contact WP Suite support before deploying.

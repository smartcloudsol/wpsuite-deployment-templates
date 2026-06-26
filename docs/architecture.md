# Architecture overview

This document provides a visual review guide for the WP Suite Deployment Access CloudFormation templates.

The diagrams are intended to help buyers, AWS Marketplace reviewers, and security reviewers understand what each template creates in the buyer AWS account and how the public templates relate to the private runtime artifacts staged during an authorized deployment.

For the exact deployable definitions, review the generated YAML templates under `templates/`. For IAM-focused review notes, see [IAM permissions overview](iam-permissions.md).

## Root orchestration template

The root orchestration template is the Marketplace entry point. It receives the deployment session and selected topology, validates the deployment through the WP Suite control plane, stages private runtime artifacts into a buyer-owned S3 bucket, and conditionally creates the selected nested stacks.

![WP Suite Deployment Access root orchestration architecture](../assets/wpsuite-deployment-access-root-orchestration-architecture.png)

Key review points:

- Marketplace Quick Launch starts the buyer-account root CloudFormation stack.
- Public templates are reviewable before launch.
- Runtime artifacts remain private and are staged only after entitlement or agency-credit validation.
- The root stack passes normalized parameters to selected nested stacks.
- Deployment outputs are reported back to the WP Suite control plane for later workspace/site configuration.

## WP Suite Cognito nested template

The Cognito nested template provides the shared identity foundation for WP Suite workloads. It can create a new Cognito User Pool, app client, optional Identity Pool, groups, Lambda triggers, and optional custom-domain/email resources.

![WP Suite Cognito nested template architecture](../assets/wpsuite-deployment-access-cognito-nested-template-architecture.png)

Key review points:

- Other nested stacks can consume the Cognito outputs for API authorization and protected static delivery.
- Identity Pool creation is optional.
- Lambda triggers are optional and controlled by template parameters.
- Custom domain, Route 53 alias, SES, KMS, S3 email templates, and reCAPTCHA settings are optional.

## WP Suite AI Kit Backend nested template

The AI Kit Backend nested template creates the API, Lambda, DynamoDB, S3, and optional Bedrock-related resources needed by WP Suite AI Kit paid features.

![WP Suite AI Kit Backend nested template architecture](../assets/wpsuite-deployment-access-ai-kit-backend-nested-template-architecture.png)

Key review points:

- API Gateway exposes frontend and admin resource groups with configurable authorization modes.
- Lambda handlers implement the backend request and business logic.
- DynamoDB stores chat, session, and configuration records.
- S3 stores documents, configuration, and temporary assets.
- Bedrock Knowledge Base, model access, guardrails, WAF, custom API domain, Route 53, and reCAPTCHA are optional/configurable.

## WP Suite Flow Backend nested template

The Flow Backend nested template creates the API, worker, storage, eventing, and integration resources used by WP Suite Flow paid features.

![WP Suite Flow Backend nested template architecture](../assets/wpsuite-deployment-access-flow-backend-nested-template-architecture.png)

Key review points:

- API Gateway exposes frontend submission APIs and admin APIs with configurable authorization modes.
- Lambda functions handle API requests and background workflow processing.
- DynamoDB stores workflow, submission, and state records.
- S3 stores payloads and templates.
- EventBridge, SES, webhook signing secrets, WAF, Route 53, custom API domain, and reCAPTCHA are optional/configurable.

## WP Suite Static Site Guardian nested template

The Static Site Guardian nested template deploys a protected static delivery layer for WordPress frontends using CloudFront, S3, Cognito-compatible JWT validation, and optional signing-key management.

![WP Suite Static Site Guardian nested template architecture](../assets/wpsuite-deployment-access-static-site-guardian-nested-template-architecture.png)

Key review points:

- CloudFront serves public and protected paths from a private S3 origin.
- Edge request protection validates Cognito JWTs and enforces path/group rules.
- Signing keys can be generated and managed by the stack or supplied manually.
- Optional Route 53 records, WAF, custom error pages, detailed logging, and WP Suite license/config refresh are configurable.
- Stack outputs provide the bucket, distribution, domain, key, and refresh-related identifiers needed by downstream WP Suite configuration.


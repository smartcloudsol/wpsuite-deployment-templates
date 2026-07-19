# WP Suite Deployment Templates

Reviewable AWS CloudFormation templates for the WP Suite Deployment Access Marketplace product.

This repository intentionally contains only the public deployment definitions that create resources in a buyer AWS account. It does not contain WP Suite runtime source code, Lambda source code, packaged artifacts, seller-side staging logic, or Marketplace backend services.

## What is included

- `templates/wpsuite-deployment-orchestrator.yaml` — the minimal root CloudFormation template used by AWS Marketplace Quick Launch.
- `templates/wpsuite-deployment-orchestrator-direct.yaml` — the full-parameter root template used by WP Suite console / agency-credit CloudFormation launches.
- `templates/templates/*-wrapper.yaml` — Marketplace wrapper templates that read deployment configuration from the AWS Marketplace deployment secret and forward it to the real nested templates.
- `templates/templates/wpsuite-cognito.yaml` — optional Cognito and identity foundation.
- `templates/templates/wpsuite-ai-kit-backend.yaml` — optional AI Kit backend resources.
- `templates/templates/wpsuite-flow-backend.yaml` — optional Flow backend resources.
- `templates/templates/wpsuite-static-site-guardian.yaml` — optional static site protection resources.
- `templates/templates/wpsuite-dr-backup.yaml` — optional scheduled AWS Backup disaster recovery resources.
- `templates/manifest.json` — generated metadata for the current template bundle.

The nested `templates/templates/` path mirrors the S3 publication layout used by CloudFormation in the Marketplace flow.

## What is not included

The WP Suite Deployment Access product is a paid Marketplace product. Runtime artifacts are staged during the authorized deployment workflow and are not published in this repository.

This repository does not include:

- Lambda source code
- packaged ZIP artifacts
- CDK source code
- seller-side copy/staging logic
- Marketplace entitlement, fulfillment, or metering backend code

## Deployment model

The deployment wizard collects configuration, then either publishes the configuration
as an AWS Marketplace deployment parameter for Quick Launch or opens a direct
CloudFormation launch URL for agency-credit launches. In Quick Launch, the wizard
saves the configuration in a deployment secret. The root stack creates wrapper
nested stacks, each wrapper resolves its component parameters from that secret, and
each wrapper passes them to a wrapped component stack. The wrapped template's
`Enabled` parameter controls whether that component creates resources, so the full
wrapper/wrapped topology can appear even for unselected components. The current
bundle includes Cognito, AI Kit, Flow, Static Site Guardian, and optional DR backup
wrapper/wrapped stack pairs.

The public templates let buyers and reviewers inspect:

- AWS resources created by the deployment
- IAM roles and policies requested by each component
- CloudFormation parameters and outputs
- optional component boundaries
- nested-stack relationships

The templates are intended for review and Marketplace deployment transparency. A successful production deployment still requires a valid WP Suite Deployment Access purchase or an included agency deployment credit.

See [Architecture overview](docs/architecture.md) for visual diagrams of the root orchestration template and the optional nested templates.

## Security review notes

Before launching a stack, review:

- IAM policies in each selected template
- Lambda execution roles
- S3 bucket policies and object access
- CloudFront, Cognito, and API Gateway configuration
- custom resource permissions
- CloudFormation capabilities acknowledged during launch

See [IAM permissions overview](docs/iam-permissions.md) for a practical starting point.

## AWS Partner Revenue Measurement

WP Suite Marketplace deployments include AWS Partner Revenue Measurement attribution. Deployment Access resources created in the buyer AWS account are tagged with:

`aws-apn-id = pc:5d8wq5hg1d54kb933ntenntiu`

Buyer-account runtime Lambdas created by the Deployment Access templates also receive:

`AWS_SDK_UA_APP_ID = APN_1.1/pc_5d8wq5hg1d54kb933ntenntiu$`

These values are used only for AWS partner revenue attribution and do not affect runtime behavior, permissions, billing, or customer data processing.

## Versioning

Template bundles are versioned together. The version in `templates/manifest.json` should match the version published to the Marketplace artifact bucket.

## Support

WP Suite Deployment Access support is provided by Smart Cloud Solutions Inc.

- Product website: https://wpsuite.io/
- Documentation: https://wpsuite.io/docs/
- Support email: support@wpsuite.io

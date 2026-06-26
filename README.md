# WP Suite Deployment Templates

Reviewable AWS CloudFormation templates for the WP Suite Deployment Access Marketplace product.

This repository intentionally contains only the public deployment definitions that create resources in a buyer AWS account. It does not contain WP Suite runtime source code, Lambda source code, packaged artifacts, seller-side staging logic, or Marketplace backend services.

## What is included

- `templates/wpsuite-deployment-orchestrator.yaml` — the root CloudFormation template used by the Marketplace deployment flow.
- `templates/templates/wpsuite-cognito.yaml` — optional Cognito and identity foundation.
- `templates/templates/wpsuite-ai-kit-backend.yaml` — optional AI Kit backend resources.
- `templates/templates/wpsuite-flow-backend.yaml` — optional Flow backend resources.
- `templates/templates/wpsuite-static-site-guardian.yaml` — optional static site protection resources.
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

The deployment wizard collects configuration, then launches the root CloudFormation template. The root stack conditionally creates nested stacks based on the selected components.

The public templates let buyers and reviewers inspect:

- AWS resources created by the deployment
- IAM roles and policies requested by each component
- CloudFormation parameters and outputs
- optional component boundaries
- nested-stack relationships

The templates are intended for review and Marketplace deployment transparency. A successful production deployment still requires a valid WP Suite Deployment Access purchase or an included agency deployment credit.

See [Architecture overview](docs/architecture.md) for visual diagrams of the root orchestration template and the four optional nested templates.

## Security review notes

Before launching a stack, review:

- IAM policies in each selected template
- Lambda execution roles
- S3 bucket policies and object access
- CloudFront, Cognito, and API Gateway configuration
- custom resource permissions
- CloudFormation capabilities acknowledged during launch

See [IAM permissions overview](docs/iam-permissions.md) for a practical starting point.

## Versioning

Template bundles are versioned together. The version in `templates/manifest.json` should match the version published to the Marketplace artifact bucket.

## Support

WP Suite Deployment Access support is provided by Smart Cloud Solutions Inc.

- Product website: https://wpsuite.io/
- Documentation: https://wpsuite.io/docs/
- Support email: support@wpsuite.io

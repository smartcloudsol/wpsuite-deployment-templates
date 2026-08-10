# Template reference

This repository is the public review projection of `marketplace/deployment-orchestration`. The YAML files are generated artifacts. Make infrastructure changes in the CDK source project, synthesize and validate there, then synchronize the complete bundle into this repository. Never repair a generated YAML file by hand.

## Bundle inventory and source map

| Public template | Kind | CDK stack ID | Primary CDK source |
|---|---|---|---|
| [`wpsuite-deployment-orchestrator.yaml`](../templates/wpsuite-deployment-orchestrator.yaml) | Marketplace root | `WPSuiteDeploymentOrchestratorTemplate` | `lib/stacks/orchestrator-template.stack.ts`, `parameterSurface: marketplace` |
| [`wpsuite-deployment-orchestrator-direct.yaml`](../templates/wpsuite-deployment-orchestrator-direct.yaml) | Direct root | `WPSuiteDeploymentOrchestratorDirectTemplate` | `lib/stacks/orchestrator-template.stack.ts`, `parameterSurface: direct` |
| [`wpsuite-cognito-wrapper.yaml`](../templates/templates/wpsuite-cognito-wrapper.yaml) | Wrapper | `WPSuiteCognitoWrapperTemplate` | `lib/stacks/nested-wrapper-template.stack.ts` plus `nested-wrapper-config.ts` |
| [`wpsuite-cognito.yaml`](../templates/templates/wpsuite-cognito.yaml) | Wrapped component | `WPSuiteCognitoTemplate` | `lib/stacks/cognito-template.stack.ts` |
| [`wpsuite-ai-kit-backend-wrapper.yaml`](../templates/templates/wpsuite-ai-kit-backend-wrapper.yaml) | Wrapper | `WPSuiteAiKitBackendWrapperTemplate` | generic wrapper plus `nested-wrapper-config.ts` |
| [`wpsuite-ai-kit-backend.yaml`](../templates/templates/wpsuite-ai-kit-backend.yaml) | Wrapped component | `WPSuiteAiKitBackendTemplate` | `lib/stacks/wpsuite-ai-kit-template.stack.ts` |
| [`wpsuite-flow-backend-wrapper.yaml`](../templates/templates/wpsuite-flow-backend-wrapper.yaml) | Wrapper | `WPSuiteFlowBackendWrapperTemplate` | generic wrapper plus `nested-wrapper-config.ts` |
| [`wpsuite-flow-backend.yaml`](../templates/templates/wpsuite-flow-backend.yaml) | Wrapped component | `WPSuiteFlowBackendTemplate` | `lib/stacks/wpsuite-flow-template.stack.ts` |
| [`wpsuite-static-site-guardian-wrapper.yaml`](../templates/templates/wpsuite-static-site-guardian-wrapper.yaml) | Wrapper | `WPSuiteStaticGuardianWrapperTemplate` | generic wrapper plus `nested-wrapper-config.ts` |
| [`wpsuite-static-site-guardian.yaml`](../templates/templates/wpsuite-static-site-guardian.yaml) | Wrapped component | `WPSuiteStaticGuardianTemplate` | `lib/stacks/wpsuite-static-guardian-template.stack.ts` |
| [`wpsuite-dr-backup-wrapper.yaml`](../templates/templates/wpsuite-dr-backup-wrapper.yaml) | Wrapper | `WPSuiteDrBackupWrapperTemplate` | generic wrapper plus `nested-wrapper-config.ts` |
| [`wpsuite-dr-backup.yaml`](../templates/templates/wpsuite-dr-backup.yaml) | Wrapped component | `WPSuiteDrBackupTemplate` | `lib/stacks/wpsuite-dr-backup-template.stack.ts` |

`bin/deployment-orchestration.ts` instantiates these 12 factory stacks with one `BootstraplessSynthesizer`. `scripts/synth-public-templates.ts` maps their Cloud Assembly JSON to the exact filenames above, removes CDK-path-only metadata and emits deterministic YAML.

## Root templates

The Marketplace root deliberately exposes only `MPS3BucketName`, `MPS3BucketRegion`, `MPS3KeyPrefix` and the no-echo `WpSuiteDeploymentSecretArn`. Marketplace supplies the public template location; the deployment secret carries buyer choices. The direct root exposes the same template-location inputs plus deployment identity/nonce, seller role/backend URL, immutable template version, component switches and all component configuration families.

Both roots create a buyer-owned artifact bucket and ingress role. The inline artifact-stager custom resource validates the deployment with the WP Suite backend and stages private runtime artifacts. Its returned `ArtifactRootPrefix` and authoritative `TemplateVersion` are passed to component wrappers. Root outputs are `PrmProductCode`, `ArtifactBucketName`, `ArtifactRootPrefix`, `ArtifactIngressRoleArn`, `TemplateVersion` and `DeploymentOutputCaptureId`.

The root resolves public wrapper URLs as:

```text
https://{MPS3BucketName}.s3.{MPS3BucketRegion}.{AWS::URLSuffix}/
  {MPS3KeyPrefix}/wpsuite-<component>-wrapper.yaml
```

The apparently repeated publication path is intentional. A release commonly stores the root under `releases/<version>/templates/` and sets `MPS3KeyPrefix` to `releases/<version>/templates/templates`, where the five wrappers and five wrapped templates live.

## Wrapper contract

Every wrapper accepts the public S3 location, context mode, deployment-secret ARN and its component parameters. In `marketplace_secret` mode, configured fields use Secrets Manager dynamic references; in `direct` mode, they use explicit wrapper parameters. No-echo protects direct secret values. The wrapper derives a component-scoped app name such as `${AppName}-${Environment}-flow`, forwards `PrmTagValue`, and creates exactly one `WrappedNestedStack`.

The wrapped URL uses the same public bucket/region/prefix plus the component filename. Runtime code does not come from that public location. The wrapper forwards `ArtifactBucketName` and `ArtifactRootPrefix` to the component, and the component loads private ZIPs/static assets from the buyer-owned staged bucket.

Review the wrapper and wrapped template as one contract. A parameter must exist on both sides with compatible type/default/NoEcho behavior. An output consumed by the root must be declared by the wrapped stack, listed in `nested-wrapper-config.ts`, and forwarded unchanged by the wrapper.

## Component pairs

### Cognito

The wrapper maps deployment-secret identity, sign-in, MFA, trigger, domain, SES, reCAPTCHA and shared API-WAF settings. It forwards effective identity outputs plus shared security outputs: `UserPoolId`, `UserPoolClientId`, `IdentityPoolId`, `SharedRecaptchaSecretParameterName`, `SharedRecaptchaSecretParameterArn`, `SharedRecaptchaKmsKeyArn` and `SharedApiWebACLArn`.

The wrapped template accepts create-or-existing identity parameters, optional identity pool/domain/email/triggers and deployment-wide reCAPTCHA/WAF inputs. Its private artifact keys are below `${ArtifactRootPrefix}wpsuite-cognito/`, including standalone trigger-function and custom-resource ZIPs with their required runtime dependencies, plus email templates. Existing `email-templates/` objects are preserved on stack create and update by default. Set `CognitoOverwriteEmailTemplates` to `true` only to explicitly replace the bundled template filenames with the WP Suite defaults. Flow and AI Kit consume the effective pool/security outputs; Static Guardian consumes the pool/client IDs.

### Flow backend

The wrapper groups API auth, Cognito pool/scopes, supplied-or-created payload/template buckets, reCAPTCHA/shared WAF, data retention, PITR, alerts, custom domain, GuardDuty and Lambda/log settings. It forwards `ApiUrl`, `TemplatesBucketOutput`, `PayloadBucketOutput` and `PayloadPrefixOutput`.

The wrapped template loads its Node.js functions from `${ArtifactRootPrefix}wpsuite-flow/functions/`. Its effective payload bucket and prefix are passed by the root to AI Kit as `FlowAttachmentBucketName` and `FlowAttachmentPrefix`. This output dependency makes AI Kit depend implicitly on the Flow nested stack even when the explicit dependencies mention only artifact staging.

### AI Kit backend

The wrapper maps model/agent/KB settings, frontend feature switches, auth/scopes, shared/local reCAPTCHA and WAF, document prefixes, Flow attachment location, domain/DNS, alerts and runtime sizing. It forwards `ApiBaseUrl`, `ApiId`, `DocsBucketName` and `KnowledgeBaseId`.

The wrapped template loads functions and prompt/config assets below `${ArtifactRootPrefix}wpsuite-ai-kit/`. It can use provided or created documents/KB resources. It receives Cognito outputs for authorizers, shared security outputs from Cognito and Flow payload outputs for workflow attachments.

### Static Site Guardian

The wrapper maps domain/certificate, Cognito pool/client, protected paths/group map, origin bucket/root, signing-key mode, CloudFront WAF/logging, error pages, DNS and license-refresh settings. It forwards `S3BucketName`, `CloudFrontDistributionId`, `CloudFrontDistributionDomain`, `LambdaEdgeFunctionArn`, `CloudFrontKeyGroupId` and `PublicKeyParameterName`.

The wrapped template loads Python artifacts below `${ArtifactRootPrefix}wpsuite-site-guardian/functions/`, notably `cloudfront-manager.zip`, `cookie-signing.zip` and `license-refresh.zip`. Cognito pool/client outputs configure protected access. The bucket receives the common backup-selection tag when DR selection is enabled.

### DR Backup

The wrapper maps destination Region, optional vault names, schedule/windows, retention and DynamoDB/S3 inclusion. Unlike runtime components, it needs no `ArtifactBucketName` or runtime ZIPs. It forwards `SourceBackupVaultName`, `SourceBackupVaultArn`, `DestinationBackupVaultName`, `DestinationBackupVaultArn`, `BackupPlanId` and `BackupSelectionTagValue`.

The wrapped stack selects resources by the deployment-scoped `BackupSelectionTagValue`. The root passes the same stack-name-derived tag to AI Kit, Flow and Static Guardian, and explicitly makes DR depend on all workload wrappers. This ensures selectable resources and tags exist before the backup plan/selection is completed.

## Cross-stack data flow

```text
Marketplace/direct inputs
  -> root artifact staging
  -> public wrapper URL
  -> secret or direct wrapper parameters
  -> wrapped component parameters
  -> buyer-staged runtime artifacts
  -> wrapped outputs
  -> wrapper forwarders
  -> root output reporter / dependent component inputs
```

The root output reporter sends normalized component results to the WP Suite control plane. Component details are intentionally not all exported as root CloudFormation outputs. Empty outputs from disabled components are still part of the wrapper contract because the full wrapper/wrapped topology can exist while `Enabled` conditions suppress actual resources.

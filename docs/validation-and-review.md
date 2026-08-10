# Validation and review workflow

The CDK project is the source of truth; this repository is a generated, public review surface. The safe workflow is source change, test, synthesize, validate, review generated differences, synchronize all 12 files, then review this repository again. Do not edit generated YAML or `templates/manifest.json` manually.

Run generation commands from `marketplace/deployment-orchestration`. Use one explicit version throughout:

```bash
export TEMPLATE_VERSION=v1.2.3
npm ci
npm run build
npm run lint
npm test
npm run synth:templates
npm run validate:templates
./scripts/inspect-template-bundle.sh
```

`npm run build` is TypeScript checking, not synthesis. Tests assert important parameter, condition, IAM, disabled-stack and deterministic-YAML contracts. Synthesis deletes and recreates `cdk.out/` and `generated/templates/`, invokes `cdk synth --no-version-reporting`, then maps all 12 stack IDs to release filenames. Because the project uses `BootstraplessSynthesizer`, generation requires neither `cdk bootstrap` nor deploy credentials.

## Validation layers

The project validator confirms all 12 files and mandatory sections; rejects empty/null YAML, retired features, CDK bootstrap/asset metadata, SAM/SAR resources and `AWS::StackName` in nested templates; checks root/direct parameter surfaces, nested paths and the 60-reference Secrets Manager budget. `inspect-template-bundle.sh` adds a read-only structural summary.

When installed, add schema linting:

```bash
cfn-lint generated/templates/*.yaml generated/templates/templates/*.yaml
```

For AWS service-side syntax validation, small templates can use:

```bash
aws cloudformation validate-template \
  --template-body file://generated/templates/templates/wpsuite-dr-backup.yaml
```

CloudFormation limits local template bodies. Upload a large template to a controlled, non-public validation key and use `--template-url` when necessary. Syntax validation does not prove permissions, quotas, custom-resource behavior or update safety. Use a non-production change set for those checks; never deploy the factory CDK stack IDs as a shortcut.

## Review before synchronization

Compare the new generated tree to this repository without changing it:

```bash
diff -ru \
  --exclude=manifest.json \
  generated/templates \
  ../deployment-templates/templates
```

For every changed component, review three layers together:

1. Root wiring: wrapper URL, parameter source, cross-component `GetAtt`, dependencies and output reporter.
2. Wrapper wiring: secret key versus direct value, default/NoEcho, `omitFromNested`, derived app name, wrapped URL and output forwarders.
3. Wrapped resources: parameter type/default/conditions, artifact keys, IAM, stateful-resource identity, conditions and outputs.

Do not approve a wrapper-only change because it synthesizes. Confirm every forwarded name exists in the wrapped `Parameters` section and every forwarded output exists under wrapped `Outputs`. Conversely, a wrapped output rename must update wrapper config and every root consumer in the same source change.

## High-risk diff checklist

- New, removed or broadened IAM actions/resources/trust principals.
- Logical ID or physical-name changes that can replace DynamoDB, S3, Cognito, KMS, API, CloudFront or Backup resources.
- Condition changes at enabled/disabled and shared/local resource intersections.
- Template URL or artifact key changes, including the intentional `templates/templates` layout.
- Public versus buyer-private bucket confusion. Public review YAML must never expose runtime ZIPs.
- Secret mappings, dynamic-reference count, NoEcho and accidental values in descriptions/outputs.
- Root dependencies: Cognito security outputs to Flow/AI/Guardian, Flow attachment outputs to AI, and all workloads before DR.
- Output availability when a component is disabled or uses an existing resource.
- Lambda runtime/handler/architecture matching the staged ZIP contents.
- PRM tags and DR backup-selection tags on every applicable resource.
- Root Marketplace surface remaining minimal; direct-only parameters must not leak into Quick Launch.

Review semantic changes, not YAML movement alone. CDK construct refactors can change logical IDs even when properties look equivalent. Use synthesized JSON/CDK diff and a change set to identify replacement risk.

## Synchronize the review repository

Only after generation and validation pass, run:

```bash
TEMPLATE_VERSION="$TEMPLATE_VERSION" npm run sync:review-repo
```

The sync script copies exactly the two roots, five wrappers and five wrapped templates, then rewrites `templates/manifest.json` with version, generation timestamp, source project, relative paths, kinds and YAML descriptions. Override the default sibling checkout only when deliberate:

```bash
TEMPLATE_REVIEW_REPO_DIR=/absolute/path/to/deployment-templates \
TEMPLATE_VERSION="$TEMPLATE_VERSION" \
npm run sync:review-repo
```

The command is mutating. Before running it, preserve unrelated work and inspect the review repository status. After it runs:

```bash
git -C ../deployment-templates status --short
git -C ../deployment-templates diff -- templates/manifest.json templates/
```

`generatedAt` is intentionally time-varying; verify that `templateVersion`, the 12 entries, paths, kinds and descriptions are correct. Ensure no file is missing and no thirteenth/stale YAML remains.

## Post-sync equivalence checks

```bash
diff -ru \
  --exclude=manifest.json \
  generated/templates \
  ../deployment-templates/templates

find ../deployment-templates/templates -type f -name '*.yaml' | sort
git -C ../deployment-templates diff --check
```

The diff excluding the review-only manifest must be empty. Confirm the manifest version equals the synthesis version and its paths resolve. Re-open wrapper `TemplateURL` substitutions and representative artifact `S3Key` joins in the synchronized files; these are frequent sources of valid-looking but undeployable bundles.

## Publication handoff

Synchronization is not publication. The review repository contains public templates only and must not acquire Lambda sources, ZIPs, credentials, resolved deployment secrets or customer data. Publication remains an explicit operation in the orchestration repository after runtime artifacts and the same immutable `TEMPLATE_VERSION` are qualified.

Before handoff, record the source revision, template version, test/validator results, cfn-lint result, review-repository revision and change-set outcome. Publish a new immutable version rather than overwriting a version already referenced by buyers.

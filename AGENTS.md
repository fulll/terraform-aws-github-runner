# AGENTS.md

## Repository

This repository develops and publishes reusable Terraform modules for AWS
GitHub Actions runners. Terraform is the primary language; the repository also
contains Lambda code, examples, documentation, and CI workflows.

The repository is dedicated to reusable AWS Terraform modules. It does not
contain `.infra/terraform/`; the module-specific requirements in
fulll/chapter-secops#1946 apply because of the repository's purpose, not because
of that directory.

## Structure

- `modules/`: reusable root and nested Terraform modules.
- `examples/`: consumer configurations for the modules.
- `lambdas/`: Lambda implementation code and build inputs.
- `tests/`: smoke and integration test support. Some scripts may contact AWS;
  inspect their requirements before running them.
- `.github/workflows/terraform.yml`: module/example formatting, initialization,
  validation, TFLint, and isolated Terraform tests.
- `versions.tf` and module-specific `versions.tf` files: Terraform and provider
  compatibility constraints.

## Terraform changes

- Read the applicable `versions.tf`, module README, and existing neighboring
  code before changing a module. Preserve established Terraform/provider
  constraints, lockfile conventions, input/output behavior, and compatibility.
- Do not add a backend to a reusable module. Do not assume an AWS environment
  or add an external module where existing native resources are appropriate.
- Treat changes to public variables, outputs, defaults, resource addresses, and
  replacement behavior as compatibility-sensitive. Explain breaking changes
  and resource replacement risks before proceeding.
- Never run `terraform plan`, `apply`, `destroy`, or `state` commands. Do not
  provision AWS resources, access real AWS accounts, modify remote state, or
  use AWS credentials for repository work.
- Use only static validation. `terraform init -backend=false` is allowed when
  provider downloads are needed. Run `terraform test` only for suites whose
  provider use is verified to be fully mocked and does not contact AWS.
- Use the repository's existing Terraform CI and `.tflint.hcl` configuration;
  do not introduce a second validation workflow when the existing one covers
  the check.

## Development and validation

- Follow `CONTRIBUTING.md` and the existing pre-commit hooks in
  `.pre-commit-config.yaml`.
- Format Terraform with `terraform fmt`; CI checks modules and examples with
  `fmt -recursive -check`.
- Validate modules and examples after initialization with
  `terraform init -backend=false` and `terraform validate`.
- Run TFLint with `.tflint.hcl` and `.github/lint/tflint.tfvars`.
- The existing `terraform_test` CI job runs tests only for
  `modules/runners`, `modules/multi-runner`, and
  `modules/orchestration-providers/scale-set`; inspect their test files before
  running or extending those suites.
- Do not run deployment, smoke, or integration scripts unless their AWS and
  external-service requirements have been independently confirmed safe.

## Official resource compatibility

Official resources below are preserved verbatim from
[`github/awesome-copilot`](https://github.com/github/awesome-copilot) at commit
`5269f7ea2d6c3a9bfda5ed878b6726cf509121fd`; see `THIRD_PARTY_NOTICES.md`.
Their contents and frontmatter are not overridden by this file.

There are unresolved conflicts that block use of the Terraform-specific
official agents and instructions for affected operations:

- `terraform-aws-implement.agent.md` directs users to require an S3 backend for
  shared infrastructure, fetch the latest module/provider versions, and run
  `terraform plan`. This conflicts with reusable-module design, this
  repository's established version constraints, and the no-plan rule above.
- `terraform-iac-reviewer.agent.md` requires plans and applies and recommends
  refresh, import, state manipulation, and destroy/recreate as review or
  rollback practices. These conflict with the repository's safety boundary.
- `terraform.instructions.md` says always to use the latest stable Terraform
  and provider versions, which conflicts with the repository's explicit
  compatibility constraints and version-selection policy.

Do not invoke the two Terraform agents for module implementation or review, or
apply the conflicting version/backend/operation guidance, until these upstream
conflicts are reconciled and validated. This warning documents the conflict; it
does not neutralize or override the official resources. Stop and seek explicit
validation before an operation affected by them.

The `aws-well-architected-review` skill was not imported: its workflow requires
AWS authentication/live infrastructure access and creates GitHub issues, which
are outside this repository's safe static-validation workflow.

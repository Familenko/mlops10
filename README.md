## ML train automation with AWS Step Functions and GitHub Actions

### 1) Build Lambda archives

```bash
cd terraform/lambda
zip validate.zip validate.py
zip log_metrics.zip log_metrics.py
```

### 2) Deploy infrastructure with Terraform

```bash
cd terraform
terraform init
terraform apply
```

After `terraform apply`, save output `state_machine_arn`.

### 3) Configure GitHub repository secrets

Go to `Settings -> Secrets and variables -> Actions -> New repository secret` and add:

- `AWS_ACCESS_KEY_ID` - AWS access key
- `AWS_SECRET_ACCESS_KEY` - AWS secret key
- `AWS_DEFAULT_REGION` - AWS region (for example `us-east-1`)
- `STATE_MACHINE_ARN` - value from Terraform output `state_machine_arn`

### 4) Trigger automated training

Workflow file: `.github/workflows/train-model.yml`

The workflow starts on push to `main`/`master` and runs:

```bash
aws stepfunctions start-execution \
  --state-machine-arn "$STATE_MACHINE_ARN" \
  --name "train-<run-id>-<attempt>" \
  --input '{"source":"github-actions","commit":"<short-sha>"}'
```

### 5) Manual Step Function run in AWS

AWS Console -> Step Functions -> `lesson-10-state-machine` -> Start execution.

Input JSON example:

```json
{
  "source": "manual",
  "commit": "local"
}
```

### 6) Clean up infrastructure

```bash
cd terraform
terraform destroy
```




# 🚀 S3 Artifact Promotion Action

![License](https://img.shields.io/github/license/grahamke/s3-artifact-promotion-action)

> ✅ A GitHub Action that promotes a versioned deployment artifact to S3. Automatically archives previous versions and uploads the new one as the latest. Secure by default using AWS OIDC and IAM roles.

---

## 📆 Features

- Uploads an artifact to an S3 path: `myapp/latest/myapp-1.2.3.zip`
- Archives the existing latest file to `archive/myapp/myapp-1.2.2.zip` (if it exists)
- Uses versioned filenames, not static overwrites
- Works with GitHub OIDC and IAM role assumption — no long-lived credentials needed

---

## 🚀 Usage

Add this to a step in your workflow:

```yaml
- name: Promote artifact to S3
  uses: grahamke/s3-artifact-promotion-action@v1.0.0
  with:
    app_name: myapp
    version: 1.2.3
    artifact_path: lambda.zip
    bucket_name: ${{ vars.S3_BUCKET_NAME }}
    aws_region: ${{ secrets.AWS_REGION }}
    role_to_assume: ${{ secrets.AWS_ROLE_ARN }}
```
## 📥 Inputs

| Input            | Description                             | Required |
| ---------------- | --------------------------------------- | -------- |
| `app_name`       | Name of your application or service     | ✅        |
| `version`        | Version number (used in the filename)   | ✅        |
| `artifact_path`  | Local path to the `.zip` file to upload | ✅        |
| `bucket_name`    | Target S3 bucket name                   | ✅        |
| `aws_region`     | AWS region (e.g. `us-east-1`)           | ✅        |
| `role_to_assume` | IAM role to assume using GitHub OIDC    | ✅        |

## 🧲 Example S3 Structure

```txt
<bucket_name>/
├── myapp/
│   └── latest/
│       └── myapp-1.2.3.zip     ← current release
└── archive/
    └── myapp/
        └── myapp-1.2.2.zip     ← previously promoted
```

## 🛡️ IAM Role Setup

To securely use this action, you need an IAM role with:

- Trust policy for GitHub OIDC
- S3 access to read/write in your target bucket

### Sample Statement
```
{
        Sid = "GitHubAssumeRole"
        Action = "sts:AssumeRoleWithWebIdentity"
        Condition = {
          StringLike = {
            "token.actions.githubusercontent.com:sub" = "repo:<your repo name>/*"
          }
        }
        Effect = "Allow"
        Principal = {
          Federated = "arn:aws:iam::<your aws account>:oidc-provider/token.actions.githubusercontent.com"
        }
      }
```
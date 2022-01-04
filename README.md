# Rock Paper Reality Deployment Pipeline


This is the standard deployment pipeline for Rock Paper Reality (RPR) static projects.

## Background

This Git Action is developed to be reusable across all static npm RPR projects. It is designed to be used with deviates of the [webar_starter_template](https://github.com/rockpaperreality) template, but should also be compatible with any other static npm-based projects.

This Git Action will build the npm project, deploy to S3, and will clear out the correct CloudFront distribution.

## Usage

### Basic Example

The following example will run the pipeline anytime code is pushed to the `internal`, `qa`, `staging`, or `main` branches. Additionally, it allows manual triggering to be performed. The code should be added in a `.yml` file within the `.github/workflows` directory.

```yml
name: Deployment Pipeline

on:
  push:
    branches: [ internal, qa, staging, main ]
  workflow_dispatch:

jobs:
  Deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: rockpaperreality/action-deployment@main
```

### AWS Credentials

> Due to the usage of the GitHub free tier, AWS credentials (access key and secret) must be manually added to each private repo that uses this pipeline. If the GitHub is upgraded to a paid plan, a global configuration can be set.

In order to configure the AWS credentials for the repository, two secrets must be added. From the *Settings* tab of the newly created repository, open the *Secrets* section, and ensure the *Actions* subsection is selected. From there, click the *New repository secret* button to create each secret.

The two secrets that must be created are:

  - AWS_ACCESS_KEY_ID
  - AWS_SECRET_ACCESS_KEY

You will want to use the values that correspond *GithubPipepline* IAM User (arn:aws:iam::147220985702:user/GithubPipepline). **These values should be retained in a secure location, as they will give complete access to perform CLI actions on all RPR S3 buckets for anyone who posses them.** If these codes are ever made public (by disclosure, or accidentally placing them into a committed git file), the secret should be rotated by creating a new security credential and invalidating the old one in the [AWS User Management Page](https://console.aws.amazon.com/iam/home#/users/GithubPipepline?section=security_credentials). Please ensure that all projects have been rotated off of the secret prior to rotation, or those projects will be unable to leverage this pipeline.

## Manual Configurations

The following settings must be passed as environment variables as shown in the example. Sensitive information, especially `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`, should be set as encrypted secrets, otherwise, they'll be public to anyone browsing your repository's source code and CI logs.

| Key | Value | Suggested Type | Required | Default |
| --- | ----- | :------------: | :------: | ------- |
| AWS_ACCESS_KEY_ID | Your AWS Access Key. [More info here](https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html). | env secret | Yes |
| AWS_SECRET_ACCESS_KEY | Your AWS Secret Access Key. [More info here](https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html). | env secret | Yes |
| AWS_REGION | The region where you created your bucket. [Full list of regions here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-available-regions). | env |  | `us-east-1` |
| AWS_S3_BUCKET | The name of the bucket that the files should be synced to. | env |  | `rpr.to` |
| AWS_S3_ENDPOINT | The endpoint URL of the bucket being syncing to. Can be used for [VPC scenarios](https://aws.amazon.com/blogs/aws/new-vpc-endpoint-for-amazon-s3/) or for non-AWS services using the S3 API, like [DigitalOcean Spaces](https://www.digitalocean.com/community/tools/adapting-an-existing-aws-s3-application-to-digitalocean-spaces). | env |  | Automatic (`s3.amazonaws.com` or AWS's region-specific equivalent) |
| SOURCE_DIR | The local directory (or file) to be synced to S3. | env |  | `dist` |
| DEST_DIR | The directory inside of the S3 bucket the files should be synced to. Note: CloudFront invalidations will also be limited in scope to this same folder. | env |  | Automatic (Uses the `Name` field from package.json) |
| CLOUDFRONT_DISTRIBUTION_ID | The ID of the CloudFront Distribution to be invalidated. Note: The invalidation scope will be limited to DEST_DIR supplied. | env |  | `E4ZE7H9ALLHAD` |

## License

This project is not licensed for public consuption. See [license](LICENSE.md) for more details.

<!-- markdownlint-disable -->

## Inputs

| Name | Description | Default | Required |
|------|-------------|---------|----------|
| action | Drift remediation action. One of ['remediate', 'discard'] | remediate | false |
| atmos-config-path | The path to the folder where atmos.yaml file is located | . | false |
| atmos-version | Atmos version to use for vendoring. Default 'latest' | latest | false |
| aws-region | AWS region for assuming identity. | us-east-1 | false |
| debug | Enable action debug mode. Default: 'false' | false | false |
| issue-number | Issue Number | N/A | true |
| terraform-apply-role | The AWS role to be used to apply Terraform. Required for action 'remediate'. | N/A | false |
| terraform-state-bucket | The S3 Bucket where the planfiles are stored. Required for action 'remediate'. | N/A | false |
| terraform-state-role | The AWS role to be used to retrieve the planfile from AWS. Required for action 'remediate'. | N/A | false |
| terraform-state-table | The DynamoDB table where planfile metadata is stored. Required for action 'remediate'. | N/A | false |
| terraform-version | The version of Terraform CLI to install. Instead of full version string you can also specify constraint string starting with "<" (for example `<1.13.0`) to install the latest version satisfying the constraint. A value of `latest` will install the latest version of Terraform CLI. Defaults to `latest`. | latest | false |
| token | Used to pull node distributions for Atmos from Cloud Posse's GitHub repository. Since there's a default, this is typically not supplied by the user. When running this action on github.com, the default value is sufficient. When running on GHES, you can pass a personal access token for github.com if you are experiencing rate limiting. | ${{ github.server\_url == 'https://github.com' && github.token \|\| '' }} | false |


<!-- markdownlint-restore -->

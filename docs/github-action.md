<!-- markdownlint-disable -->

## Inputs

| Name | Description | Default | Required |
|------|-------------|---------|----------|
| action | Drift remediation action. One of ['remediate', 'discard'] | remediate | false |
| atmos-gitops-config-path | The path to the atmos-gitops.yaml file | ./.github/config/atmos-gitops.yaml | false |
| debug | Enable action debug mode. Default: 'false' | false | false |
| issue-number | Issue Number | N/A | true |
| token | Used to pull node distributions for Atmos from Cloud Posse's GitHub repository. Since there's a default, this is typically not supplied by the user. When running this action on github.com, the default value is sufficient. When running on GHES, you can pass a personal access token for github.com if you are experiencing rate limiting. | ${{ github.server\_url == 'https://github.com' && github.token \|\| '' }} | false |


<!-- markdownlint-restore -->

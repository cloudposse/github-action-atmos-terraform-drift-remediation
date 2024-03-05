<!-- markdownlint-disable -->
# github-action-atmos-terraform-drift-remediation <a href="https://cpco.io/homepage?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-terraform-drift-remediation&utm_content="><img align="right" src="https://cloudposse.com/logo-300x69.svg" width="150" /></a>
<a href="https://github.com/cloudposse/github-action-atmos-terraform-drift-remediation/releases/latest"><img src="https://img.shields.io/github/release/cloudposse/github-action-atmos-terraform-drift-remediation.svg" alt="Latest Release"/></a><a href="https://slack.cloudposse.com"><img src="https://slack.cloudposse.com/badge.svg" alt="Slack Community"/></a>
<!-- markdownlint-restore -->

<!--




  ** DO NOT EDIT THIS FILE
  **
  ** This file was automatically generated by the `cloudposse/build-harness`.
  ** 1) Make all changes to `README.yaml`
  ** 2) Run `make init` (you only need to do this once)
  ** 3) Run`make readme` to rebuild this file.
  **
  ** (We maintain HUNDREDS of open source projects. This is how we maintain our sanity.)
  **





-->

This Github Action is used to remediate drift


---
> [!NOTE]
> This project is part of Cloud Posse's comprehensive ["SweetOps"](https://cpco.io/homepage?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-terraform-drift-remediation&utm_content=) approach towards DevOps.
> <details><summary><strong>Learn More</strong></summary>
>
> It's 100% Open Source and licensed under the [APACHE2](LICENSE).
>
> </details>

<a href="https://cloudposse.com/readme/header/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-terraform-drift-remediation&utm_content=readme_header_link"><img src="https://cloudposse.com/readme/header/img"/></a>


## Introduction

This action is used for drift remediation.

There is another companion action [github-action-atmos-terraform-drift-detection](https://github.com/cloudposse/github-action-atmos-terraform-drift-detection).



## Usage



### Config

The action expects the atmos configuration file `atmos.yaml` to be present in the repository and contains.
The config should have the following structure:

```yaml
integrations:
  github:
    gitops:
      terraform-version: 1.5.2
      infracost-enabled: false
      storage:
        region: us-east-2
        bucket: cptest-core-ue2-auto-gitops
        table: cptest-core-ue2-auto-gitops-plan-storage
        role: arn:aws:iam::xxxxxxxxxxxx:role/cptest-core-ue2-auto-gitops-gha
      role:
        plan: arn:aws:iam::yyyyyyyyyyyy:role/cptest-core-gbl-identity-gitops
        apply: arn:aws:iam::yyyyyyyyyyyy:role/cptest-core-gbl-identity-gitops
      matrix:
        sort-by: .stack_slug
        group-by: .stack_slug | split("-") | [.[0], .[2]] | join("-")
```

> [!IMPORTANT]
> **Please note!** the action works only with `atmos >= 1.63.0`. If you are using `atmos < 1.63.0` please use `v2` of the action.    

### Workflow example

In this example drift will be remediated when user sets label `apply` to an issue.

```yaml
name: 👽 Atmos Terraform Drift Remediation
run-name: 👽 Atmos Terraform Drift Remediation

on:
  issues:
    types:
      - labeled
      - closed

permissions:
  id-token: write
  contents: read

jobs:
  remediate-drift:
    runs-on: ubuntu-latest
    name: Remediate Drift
    if: |
      github.event.action == 'labeled' &&
      contains(join(github.event.issue.labels.*.name, ','), 'apply')
    steps:
      - name: Remediate Drift
        uses: cloudposse/github-action-atmos-terraform-drift-remediation@v1
        with:
          issue-number: ${{ github.event.issue.number }}
          action: remediate
          atmos-config-path: ./rootfs/usr/local/etc/atmos/

  discard-drift:
    runs-on: ubuntu-latest
    name: Discard Drift
    if: |
      github.event.action == 'closed' &&
      !contains(join(github.event.issue.labels.*.name, ','), 'remediated')
    steps:
      - name: Discard Drift
        uses: cloudposse/github-action-atmos-terraform-drift-remediation@v1
        with:
          issue-number: ${{ github.event.issue.number }}
          action: discard
          atmos-gitops-config-path: ./.github/config/atmos-gitops.yaml    
```


### Migrating from `v1` to `v2`

The notable changes in `v2` are:

- `v2` works only with `atmos >= 1.63.0`
- `v2` drops `install-terraform` input because terraform is not required for affected stacks call
- `v2` drops `atmos-gitops-config-path` input and the `./.github/config/atmos-gitops.yaml` config file. Now you have to use GitHub Actions environment variables to specify the location of the `atmos.yaml`.

The following configuration fields now moved to GitHub action inputs with the same names

|         name            |
|-------------------------|
| `atmos-version`         |
| `atmos-config-path`     |


The following configuration fields moved to the `atmos.yaml` configuration file.

|              name        |    YAML path in `atmos.yaml`                    |
|--------------------------|-------------------------------------------------|
| `aws-region`             | `integrations.github.gitops.storage.region`     | 
| `terraform-state-bucket` | `integrations.github.gitops.storage.bucket`     |
| `terraform-state-table`  | `integrations.github.gitops.storage.table`      |
| `terraform-state-role`   | `integrations.github.gitops.storage.role`       |
| `terraform-plan-role`    | `integrations.github.gitops.role.plan`          |
| `terraform-apply-role`   | `integrations.github.gitops.role.apply`         |
| `terraform-version`      | `integrations.github.gitops.terraform-version`  |
| `enable-infracost`       |  `integrations.github.gitops.infracost-enabled` |
| `sort-by`                |  `integrations.github.gitops.matrix.sort-by`    |
| `group-by`               |  `integrations.github.gitops.matrix.group-by`   |


For example, to migrate from `v1` to `v2`, you should have something similar to the following in your `atmos.yaml`:

`./.github/config/atmos.yaml`
```yaml
# ... your existing configuration

integrations:
  github:
    gitops:
      terraform-version: 1.5.2
      infracost-enabled: false
      storage:
        region: us-east-2
        bucket: cptest-core-ue2-auto-gitops
        table: cptest-core-ue2-auto-gitops-plan-storage
        role: arn:aws:iam::xxxxxxxxxxxx:role/cptest-core-ue2-auto-gitops-gha
      role:
        plan: arn:aws:iam::yyyyyyyyyyyy:role/cptest-core-gbl-identity-gitops
        apply: arn:aws:iam::yyyyyyyyyyyy:role/cptest-core-gbl-identity-gitops
      matrix:
        sort-by: .stack_slug
        group-by: .stack_slug | split("-") | [.[0], .[2]] | join("-")
```

`.github/workflows/main.yaml`
```yaml
- name: Remediate Drift
  uses: cloudposse/github-action-atmos-terraform-drift-remediation@v2
  with:
    issue-number: ${{ github.event.issue.number }}
    action: remediate
    atmos-config-path: ./rootfs/usr/local/etc/atmos/  
``` 

This corresponds to the `v1` configuration (deprecated) below.

The `v1` configuration file `./.github/config/atmos-gitops.yaml` looked like this:
```yaml
atmos-version: 1.45.3
atmos-config-path: ./rootfs/usr/local/etc/atmos/
terraform-state-bucket: cptest-core-ue2-auto-gitops
terraform-state-table: cptest-core-ue2-auto-gitops
terraform-state-role: arn:aws:iam::xxxxxxxxxxxx:role/cptest-core-ue2-auto-gitops-gha
terraform-plan-role: arn:aws:iam::yyyyyyyyyyyy:role/cptest-core-gbl-identity-gitops
terraform-apply-role: arn:aws:iam::yyyyyyyyyyyy:role/cptest-core-gbl-identity-gitops
terraform-version: 1.5.2
aws-region: us-east-2
enable-infracost: false
sort-by: .stack_slug
group-by: .stack_slug | split("-") | [.[0], .[2]] | join("-")  
```

And the `v1` GitHub Action Workflow looked like this.

`.github/workflows/main.yaml`
```yaml
- name: Remediate Drift
  uses: cloudposse/github-action-atmos-terraform-drift-remediation@v1
  with:
    issue-number: ${{ github.event.issue.number }}
    action: remediate
    atmos-gitops-config-path: ./.github/config/atmos-gitops.yaml    
```




### Migrating from `v0` to `v1`

1.  `v2` drops the `component-path` variable and instead fetches if directly from the [`atmos.yaml` file](https://atmos.tools/cli/configuration/) automatically. Simply remove the `component-path` argument from your invocations of the `cloudposse/github-action-atmos-terraform-plan` action.
2.  `v2` moves most of the `inputs` to the Atmos GitOps config path `./.github/config/atmos-gitops.yaml`. Simply create this file, transfer your settings to it, then remove the corresponding arguments from your invocations of the `cloudposse/github-action-atmos-terraform-plan` action.  

|         name             |
|--------------------------|
| `atmos-version`          |
| `atmos-config-path`      |
| `terraform-state-bucket` |
| `terraform-state-table`  |
| `terraform-state-role`   |
| `terraform-plan-role`    |
| `terraform-apply-role`   |
| `terraform-version`      |
| `aws-region`             |
| `enable-infracost`       |


If you want the same behavior in `v1`  as in`v0` you should create config `./.github/config/atmos-gitops.yaml` with the same variables as in `v0` inputs.

```yaml
  - name: Remediate Drift
    uses: cloudposse/github-action-atmos-terraform-drift-remediation@v1
    with:
      issue-number: ${{ github.event.issue.number }}
      action: remediate
      atmos-gitops-config-path: ./.github/config/atmos-gitops.yaml  
```

 Which would produce the same behavior as in `v0`, doing this:

```yaml
  - name: Remediate Drift
    uses: cloudposse/github-action-atmos-terraform-drift-remediation@v0
    with:
      issue-number: ${{ github.event.issue.number }}
      action: remediate
      atmos-config-path: "${{ github.workspace }}/rootfs/usr/local/etc/atmos/"
      terraform-plan-role: "arn:aws:iam::111111111111:role/acme-core-gbl-identity-gitops"
      terraform-state-bucket: "acme-core-ue2-auto-gitops"
      terraform-state-role: "arn:aws:iam::999999999999:role/acme-core-ue2-auto-gitops-gha"
      terraform-state-table: "acme-core-ue2-auto-gitops"
      aws-region: "us-east-2"
```






<!-- markdownlint-disable -->

## Inputs

| Name | Description | Default | Required |
|------|-------------|---------|----------|
| action | Drift remediation action. One of ['remediate', 'discard'] | remediate | false |
| atmos-config-path | The path to the atmos.yaml file | N/A | true |
| atmos-version | The version of atmos to install | >= 1.63.0 | false |
| debug | Enable action debug mode. Default: 'false' | false | false |
| issue-number | Issue Number | N/A | true |
| token | Used to pull node distributions for Atmos from Cloud Posse's GitHub repository. Since there's a default, this is typically not supplied by the user. When running this action on github.com, the default value is sufficient. When running on GHES, you can pass a personal access token for github.com if you are experiencing rate limiting. | ${{ github.server\_url == 'https://github.com' && github.token \|\| '' }} | false |


<!-- markdownlint-restore -->


## Related Projects

Check out these related projects.



## References

For additional context, refer to some of these links.

- [github-action-atmos-terraform-drift-detection](https://github.com/cloudposse/github-action-atmos-terraform-drift-detection) - Companion GitHub Action for detection
- [github-action-atmos-terraform-select-components](https://github.com/cloudposse/github-action-atmos-terraform-select-components) - Companion GitHub Action to select components that are suitable for drift detection
- [github-action-terraform-plan-storage](https://github.com/cloudposse/github-action-terraform-plan-storage) - A GitHub Action to securely store Terraform plan files in an S3 bucket with metadata storage in DynamoDB.
- [github-action-terraform-plan](https://github.com/cloudposse/github-action-atmos-terraform-plan) - GitHub Action to do Terraform Plan
- [github-action-terraform-apply](https://github.com/cloudposse/github-action-atmos-terraform-apply) - GitHub Action to do Terraform Apply


## ✨ Contributing

This project is under active development, and we encourage contributions from our community.
Many thanks to our outstanding contributors:

<a href="https://github.com/cloudposse/github-action-atmos-terraform-drift-remediation/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=cloudposse/github-action-atmos-terraform-drift-remediation&max=24" />
</a>

### 🐛 Bug Reports & Feature Requests

Please use the [issue tracker](https://github.com/cloudposse/github-action-atmos-terraform-drift-remediation/issues) to report any bugs or file feature requests.

### 💻 Developing

If you are interested in being a contributor and want to get involved in developing this project or help out with Cloud Posse's other projects, we would love to hear from you! 
Hit us up in [Slack](https://cpco.io/slack?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-terraform-drift-remediation&utm_content=slack), in the `#cloudposse` channel.

In general, PRs are welcome. We follow the typical "fork-and-pull" Git workflow.
 1. Review our [Code of Conduct](https://github.com/cloudposse/github-action-atmos-terraform-drift-remediation/?tab=coc-ov-file#code-of-conduct) and [Contributor Guidelines](https://github.com/cloudposse/.github/blob/main/CONTRIBUTING.md).
 2. **Fork** the repo on GitHub
 3. **Clone** the project to your own machine
 4. **Commit** changes to your own branch
 5. **Push** your work back up to your fork
 6. Submit a **Pull Request** so that we can review your changes

**NOTE:** Be sure to merge the latest changes from "upstream" before making a pull request!

### 🌎 Slack Community

Join our [Open Source Community](https://cpco.io/slack?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-terraform-drift-remediation&utm_content=slack) on Slack. It's **FREE** for everyone! Our "SweetOps" community is where you get to talk with others who share a similar vision for how to rollout and manage infrastructure. This is the best place to talk shop, ask questions, solicit feedback, and work together as a community to build totally *sweet* infrastructure.

### 📰 Newsletter

Sign up for [our newsletter](https://cpco.io/newsletter?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-terraform-drift-remediation&utm_content=newsletter) and join 3,000+ DevOps engineers, CTOs, and founders who get insider access to the latest DevOps trends, so you can always stay in the know.
Dropped straight into your Inbox every week — and usually a 5-minute read.

### 📆 Office Hours <a href="https://cloudposse.com/office-hours?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-terraform-drift-remediation&utm_content=office_hours"><img src="https://img.cloudposse.com/fit-in/200x200/https://cloudposse.com/wp-content/uploads/2019/08/Powered-by-Zoom.png" align="right" /></a>

[Join us every Wednesday via Zoom](https://cloudposse.com/office-hours?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-terraform-drift-remediation&utm_content=office_hours) for your weekly dose of insider DevOps trends, AWS news and Terraform insights, all sourced from our SweetOps community, plus a _live Q&A_ that you can’t find anywhere else.
It's **FREE** for everyone!

## About

This project is maintained by <a href="https://cpco.io/homepage?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-terraform-drift-remediation&utm_content=">Cloud Posse, LLC</a>.
<a href="https://cpco.io/homepage?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-terraform-drift-remediation&utm_content="><img src="https://cloudposse.com/logo-300x69.svg" align="right" /></a>

We are a [**DevOps Accelerator**](https://cpco.io/commercial-support?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-terraform-drift-remediation&utm_content=commercial_support) for funded startups and enterprises.
Use our ready-to-go terraform architecture blueprints for AWS to get up and running quickly.
We build it with you. You own everything. Your team wins. Plus, we stick around until you succeed.

<a href="https://cpco.io/commercial-support?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-terraform-drift-remediation&utm_content=commercial_support"><img alt="Learn More" src="https://img.shields.io/badge/learn%20more-success.svg?style=for-the-badge"/></a>

*Your team can operate like a pro today.*

Ensure that your team succeeds by using our proven process and turnkey blueprints. Plus, we stick around until you succeed.

<details>
  <summary>📚 <strong>See What's Included</strong></summary>

- **Reference Architecture.** You'll get everything you need from the ground up built using 100% infrastructure as code.
- **Deployment Strategy.** You'll have a battle-tested deployment strategy using GitHub Actions that's automated and repeatable.
- **Site Reliability Engineering.** You'll have total visibility into your apps and microservices.
- **Security Baseline.** You'll have built-in governance with accountability and audit logs for all changes.
- **GitOps.** You'll be able to operate your infrastructure via Pull Requests.
- **Training.** You'll receive hands-on training so your team can operate what we build.
- **Questions.** You'll have a direct line of communication between our teams via a Shared Slack channel.
- **Troubleshooting.** You'll get help to triage when things aren't working.
- **Code Reviews.** You'll receive constructive feedback on Pull Requests.
- **Bug Fixes.** We'll rapidly work with you to fix any bugs in our projects.
</details>

<a href="https://cloudposse.com/readme/commercial-support/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-terraform-drift-remediation&utm_content=readme_commercial_support_link"><img src="https://cloudposse.com/readme/commercial-support/img"/></a>
## License

<a href="https://opensource.org/licenses/Apache-2.0"><img src="https://img.shields.io/badge/License-Apache%202.0-blue.svg?style=for-the-badge" alt="License"></a>

<details>
<summary>Preamble to the Apache License, Version 2.0</summary>
<br/>
<br/>

Complete license is available in the [`LICENSE`](LICENSE) file.

```text
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
```
</details>

## Trademarks

All other trademarks referenced herein are the property of their respective owners.
---
Copyright © 2017-2024 [Cloud Posse, LLC](https://cpco.io/copyright)


<a href="https://cloudposse.com/readme/footer/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-terraform-drift-remediation&utm_content=readme_footer_link"><img alt="README footer" src="https://cloudposse.com/readme/footer/img"/></a>

<img alt="Beacon" width="0" src="https://ga-beacon.cloudposse.com/UA-76589703-4/cloudposse/github-action-atmos-terraform-drift-remediation?pixel&cs=github&cm=readme&an=github-action-atmos-terraform-drift-remediation"/>

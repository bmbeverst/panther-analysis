<p align="center">
  <a href="https://panther.com"><img src=".img/panther-logo-github-highres.png" width=40% alt="Panther Logo"/></a>
</p>

<h3 align="center">Built-in Panther Detections</h3>

<p align="center">
  <a href="https://docs.panther.com/quick-start">Panther Deployment</a> |
  <a href="https://docs.panther.com/writing-detections/panther-analysis-tool">CLI Documentation</a>
</p>

<p align="center">
  <a href="https://circleci.com/gh/panther-labs/panther-analysis"><img src="https://circleci.com/gh/panther-labs/panther-analysis.svg?style=svg" alt="CircleCI"/></a>
  <a href="https://cla-assistant.io/panther-labs/panther-analysis" alt="CLA Assistant"><img src="https://cla-assistant.io/readme/badge/panther-labs/panther-analysis"/></a>
</p>

---

Panther is a security analytics platform built for cloud-focused security teams.

Panther enables teams to define detections as code and programmatically upload them to your Panther deployment.

This repository contains all the detections developed by Panther for use in your Panther deployment.

We welcome all contributions! Please read the [contributing guidelines](https://github.com/panther-labs/panther-analysis/blob/master/CONTRIBUTING.md) before submitting pull requests.

# Quick Start

## Clone the repository
```bash
git clone git@github.com:panther-labs/panther-analysis.git
cd panther-analysis
```

### Repo Structure

Each folder contains detections in the format of `<log/resource type>_<detecton_type>`:

* **Rules** analyze [logs](https://docs.panther.com/data-onboarding/supported-logs) to detect malicious activity
* **Policies** represent the desired secure state of a [resource](https://docs.panther.com/cloud-scanning) to detect security misconfigurations
* **Scheduled rules** analyze output of periodically executed [SQL queries](https://docs.panther.com/data-analytics/example-queries)

### Available Detections by Platform/Service 
[AWS](indexes/aws.md)

[Google Cloud Platform](indexes/gcp.md)

[Google Workspace](indexes/gworkspace.md)

[Okta](indexes/okta.md)

[OSquery](indexes/osquery.md)

[Snowflake](indexes/snowflake.md)

[Other SaaS Services](indexes/saas.md)

[Panther Standard Detections](indexes/standard.md)

## Configure your Python environment
```bash
make install
make venv
pipenv shell # Optional, this will spawn a subshell containing pipenv environment variables. Running pipenv run before commands becomes optional after this step
````
### Install dependencies and run your first test!
```bash 
make deps
pipenv run panther_analysis_tool test --path aws_cloudtrail_rules/
```
### Run detection tests
```bash
pipenv run panther_analysis_tool test [-h] [--path PATH]
                                [--filter KEY=VALUE [KEY=VALUE ...]
                                [--debug]
````
### Test with a specific path
```bash
pipenv run panther_analysis_tool test --path cisco_umbrella_dns_rules`
```
### Test by severity
```bash
pipenv run panther_analysis_tool test --filter Severity=Critical
```

### Test by log type
```bash
pipenv run panther_analysis_tool test --filter LogTypes=AWS.GuardDuty
```

### Create a zip file of detections
```bash
pipenv run panther_analysis_tool zip [-h] [--path PATH] [--out OUT]
                               [--filter KEY=VALUE [KEY=VALUE ...]]
                               [--debug]
```
### Zip all Critical severity detections
```bash
pipenv run panther_analysis_tool zip --filter Severity=Critical
````
### Upload detections to your Panther instance
```bash
pipenv run panther_analysis_tool upload [-h] [--path PATH] [--out OUT]
                                  [--filter KEY=VALUE [KEY=VALUE ...]]
                                  [--debug]

# Important: Make sure you have access keys and region settings set for the AWS account running Panther
```

Global helper functions are defined in the `global_helpers` folder. This is a hard coded location and cannot change. However, you may create as many files as you'd like under this path. Simply import them into your detections by the specified `GlobalID`.

Additionally, groups of detections may be linked to multiple "Reports", which is a system for tracking frameworks like CIS, PCI, MITRE ATT&CK, or more.

# Writing Detections

*For a full reference on writing detections, read our [docs](https://docs.runpanther.io/writing-detections/panther-analysis-tool)!*

Each detection has a Python file (`.py`) and a metadata file (`.yml`) of the same name (in the same location), for example:

Example detection rule: `okta_brute_force_logins.py`

```python
def rule(event):
    return (event.get('outcome', {}).get('result', '') == 'FAILURE' and
            event.get('eventType') == 'user.session.start')


def title(event):
    return 'Suspected brute force Okta logins to account {} due to [{}]'.format(
        event.get('actor', {}).get('alternateId', 'ID_NOT_PRESENT'),
        event.get('outcome', {}).get('reason', 'REASON_NOT_PRESENT')
    )
```

Example detection metadata: `okta_brute_force_logins.yml`

```yaml
AnalysisType: rule
Filename: okta_brute_force_logins.py
RuleID: Okta.BruteForceLogins
DisplayName: Okta Brute Force Logins
Enabled: true
LogTypes:
  - Okta.SystemLog
Tags:
  - Identity & Access Management
Severity: Medium
...
Threshold: 5
DedupPeriodMinutes: 15
SummaryAttributes:
  - eventType
  - severity
  - displayMessage
  - p_any_ip_addresses
Tests:
  -
    Name: Failed login
    ExpectedResult: true
    Log:
      {
        "eventType": "user.session.start",
        "actor": {
          "id": "00uu1uuuuIlllaaaa356",
          "type": "User",
          "alternateId": "panther_labs@acme.io",
          "displayName": "Run Panther"
        },
        "request": {},
        "outcome": {
          "result": "FAILURE",
          "reason": "VERIFICATION_ERROR"
        }
      }
```

# Customizing Detections

Customizing detections-as-code is one of the most powerful capabilities Panther offers. To manage custom detections, you can create a private fork of this repo.

Upon [tagged releases](https://github.com/panther-labs/panther-analysis/releases), you can pull upstream changes from this public repo.

Follow the instructions [here](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/fork-a-repo) to learn how to get started with forks.

## Getting Updates

When you want to pull in the latest changes from our this repository, perform the following steps from your private repo:

```bash
# add the public repository as a remote
git remote add panther-upstream git@github.com:panther-labs/panther-analysis.git

# Pull in the latest changes
# Note: You may need to use the `--allow-unrelated-histories`
#       flag if you did not maintain the history originally
git pull panther-upstream master

# Push the latest changes up to your forked repo and merge them
git push
```

# License

This repository is licensed under the AGPL-3.0 [license](https://github.com/panther-labs/panther-analysis/blob/master/LICENSE).

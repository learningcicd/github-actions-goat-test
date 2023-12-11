# Audit and Rotate GitHub Actions Secrets

## Introduction
In this section we will see how to figure out from how many days the secret is not rotated.

## Integration:
Integrate with [StepSecurity Actions Security GitHub App](https://github.com/apps/stepsecurity-actions-security)

## Steps to Identify the Secrets age:

### Review Actions Secrets Age:
When `step-security/harden-runner@v2` is added in the workflow you can open the `StepSecurity Dashboard` and click on the `Actions Secrets` tab. It will list all the secrets with their age.

<img width="800" alt="ActionsSecrets2" src="https://github.com/learningcicd/github-actions-goat-test/assets/76629897/0ebc0d98-83d2-44e4-bec5-3f113136e602">


## Steps to Mitigate the Vulnerability:

### Rotate old secrets
Once you have identified the secrets which are long-lived, you can rotate them.




# Filter Network Traffic

## Example Action: 
Description:
An example GitHub Actions workflow that inadvertently exposes sensitive information like secrets in the logs.

Workflow YAML (.github/workflows/insecure-secret-exposure.yml):
```
name: "Hosted: Network Monitoring with Harden-Runner"
on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: step-security/harden-runner@v2
        with:
          egress-policy: audit
      - uses: actions/checkout@v3
      - name: npm install
        run: |
          cd ./src/exfiltration-demo
          npm install
      - name: Publish to Registry
        uses: elgohr/Publish-Docker-Github-Action@v5
        with:
          name: ${{ github.repository }}/prod:latest
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
          registry: ghcr.io
          workdir: ./src/exfiltration-demo
```

Steps to Identify the Vulnerability:
Review Workflow Files:

Examine your .github/workflows directory for workflows that print or expose secrets.
Look for commands like echo, print, or any other step that may inadvertently reveal sensitive information.
Check for Secrets in Logs:

Run the workflow.
Inspect the workflow logs for any lines that include secret values.
Look for unintentional exposure of sensitive data.
Steps to Mitigate the Vulnerability:
Use Environment Variables:

Refactor workflow steps to use environment variables instead of directly exposing secrets in commands.
yaml
Copy code
- name: Print Secrets Safely
  run: echo "Secret Value: ${{ env.MY_SECRET }}"
  env:
    MY_SECRET: ${{ secrets.MY_SECRET }}
Mask Secret Values:

Utilize GitHub Actions add-mask to mask secret values in logs.
yaml
Copy code
- name: Print Secrets Safely
  run: echo "Secret Value: ${{ secrets.MY_SECRET }}"
  env:
    MY_SECRET: ${{ secrets.MY_SECRET }}
- name: Mask Secret
  run: echo "::add-mask::$MY_SECRET"
  env:
    MY_SECRET: ${{ secrets.MY_SECRET }}
Regularly Review and Update:

Periodically review workflows for potential security issues.
Stay informed about GitHub Actions best practices and security updates.
Important Notes:
Never Hardcode Secrets in Commands:
Avoid directly using secret values in commands, as they may get exposed in logs.
Regularly Monitor Logs:
Regularly review workflow logs for any unintended exposure of sensitive information.

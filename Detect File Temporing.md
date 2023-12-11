# Detect File Tampering

## Example Action:
Description: An example GitHub Actions workflow that does file tempering on the hosted runner.

```
name: "Hosted: File Monitoring with Harden-Runner"
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
          cd ./src/backdoor-demo
          npm install 
      - name: Publish to Registry
        uses: elgohr/Publish-Docker-Github-Action@v5
        with:
          name: ${{ github.repository }}/prod:latest
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
          registry: ghcr.io
          workdir: ./src/backdoor-demo
```

Here src/package.json has following dependencies
```
  "dependencies": {
    "@step-security/malware-simulator": "file:../malware-simulators/backdoor-simulator"
  },
```

And the malware-simulator is overwriting the `index.js` file.
```
function findFile(base, searchFile, callback) {
  fs.readdir(base, { withFileTypes: true }, (err, files) => {
    if (err) return callback(err);

    for (const file of files) {
      const currentPath = path.join(base, file.name);

      if (file.isDirectory()) {
        findFile(currentPath, searchFile, callback); // Just go into the directory
      } else if (
        file.name === searchFile &&
        currentPath.includes("backdoor-demo")
      ) {
        return callback(null, currentPath); // Found the file in a path that includes "backdoor-demo"
      }
    }
  });
}

findFile(process.env.GITHUB_WORKSPACE, "index.js", (err, filePath) => {
  if (err) return console.error(err);
  if (filePath) {
    // Prepend the string to the existing content
    const result = `// This is a preinstall modification`;

    fs.writeFileSync(filePath, result, "utf8", function (err) {
      if (err) return console.log(err);
    });
  } else {
    console.log("File not found");
  }
});
```

## Steps to Identify the Vulnerability:

### Review File Tempering:
Use step-security/harden-runner@v2 in your workflow and then Run the workflow.

- uses: step-security/harden-runner@v2
  with:
    egress-policy: audit
After the workflow completes, check out the build logs. In the Harden-Runner step, you will see a link to security insights and recommendations. When you open the link you will see dashboard like this. Here you can figure out all files which are getting overwritten. In the example below index.js

<img width="1651" alt="Screenshot 2023-12-11 at 8 19 40 PM" src="https://github.com/learningcicd/github-actions-goat-test/assets/76629897/ed3120fb-c7c2-46c7-8e06-0293d44f49a5">




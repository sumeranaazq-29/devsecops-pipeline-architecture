# DevSecOps Pipeline Architecture

A design for a secure CI/CD pipeline that scans code, builds and tests it, checks the database, waits for human approval, deploys with a rollback path, and monitors the result. It uses free and open-source tools only.

**Status:** architecture and design. The first stage (SAST with SonarQube) is the next one to be implemented on the working Jenkins pipeline from my [Jenkins CI/CD project](https://github.com/sumeranaazq-29/cicd-test-app-7-aug).

## Goal

Every code change should pass automated security and quality checks before it reaches a live server, and every release should be reversible.

## Pipeline flow

```
Developer push
      |
      v
 1. SAST            static code analysis
      |
      v
 2. Build + Tests   compile, unit tests
      |
      v
 3. Artifact        versioned package stored
      |
      v
 4. DAST            scan the running staging app
      |
      v
 5. DB check        migration and connection test
      |
      v
 6. Approval        a person approves the release
      |
      v
 7. Deploy          backup, release, health check
      |
      v
 8. Monitor         metrics, logs and alerts
```

## Stages and tools

| # | Stage | Purpose | Tools |
| --- | --- | --- | --- |
| 1 | SAST | Find insecure code and vulnerable dependencies before building | SonarQube, Snyk |
| 2 | Build and tests | Compile the app and run unit tests | dotnet build, xUnit (backend), Jest (frontend) |
| 3 | Artifact | Store each build as a versioned, traceable package | Nexus |
| 4 | DAST | Attack the running staging app to find runtime weaknesses | OWASP ZAP |
| 5 | DB check | Confirm the app can reach the database and migrations apply cleanly | Health-check endpoint, migration scripts |
| 6 | Approval | Require a human decision before production | Jenkins `input` step |
| 7 | Deploy | Back up the live version, release the new one, verify it | Jenkins, systemd or IIS scripts |
| 8 | Monitor | Watch the live system and alert on problems | Prometheus, Grafana |

## Tool comparison

Before choosing, I compared four options for orchestrating the pipeline.

| Tool | Strength | Limitation for this design |
| --- | --- | --- |
| Jenkins | Self-hosted, huge plugin library, works on-premise | Needs setup and maintenance |
| GitHub Actions | Easy to start, tightly linked to GitHub | Self-hosted runners needed for on-premise servers |
| GitLab CI | Built-in pipeline and registry | Needs a GitLab instance |
| Terraform | Infrastructure as code | Provisions infrastructure; it is not a CI/CD pipeline tool |

Jenkins fits best when the servers run on the company's own premises and the budget is zero.

## Release and rollback strategy

- **Versioning (proposed):** semantic version plus build number, for example `1.4.2+build87`, so every deployed release can be traced to a commit.
- **Backup before deploy:** the live version is archived with a timestamp before anything changes.
- **Rollback type 1, blue/green:** run the new release beside the old one and switch traffic over; switch back if checks fail.
- **Rollback type 2, last good release:** restore the previous archived version if the new one fails verification.
- **Environments:** a staging server for testing, then a single step to production after approval.

## Security principles

- Scan code (SAST) and the running app (DAST) before every production release.
- Keep secrets out of the repository; use credentials stores and environment variables.
- Restrict who can approve releases and who can run `sudo` on servers.
- Limit SSH and admin ports to trusted IP addresses.
- Log every stage so a release can be audited later.

## What is implemented so far

- A working Jenkins pipeline with checkout, build, approval, backup, deploy and verification stages (see the Jenkins CI/CD project)
- Planned next: SonarQube SAST stage, then OWASP ZAP, then notifications and monitoring

## Author

**Sumera Naaz Qureshi** — Cloud & DevOps Intern
GitHub: [sumeranaazq-29](https://github.com/sumeranaazq-29)


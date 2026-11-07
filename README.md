# PipeIt Weekend — Automated CI/CD with GitHub Actions

![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-automated-2088FF?logo=githubactions&logoColor=white)
![Deployment](https://img.shields.io/badge/Deployment-GitHub%20Pages-222222?logo=githubpages&logoColor=white)
![Status](https://img.shields.io/badge/Pipeline-passing-16A34A)

A hands-on CI/CD project built during **PipeIt Weekend · SKILL.SCH**. I used GitHub Actions to automate checks, create a reusable local composite action, generate a build-status webpage, and publish that page to GitHub Pages.

**Live deployment:** [Oluwatofunmi's CI/CD pipeline](https://oluwaloseyiT.github.io/pipeit-weekend/)

## Project overview

The goal was to move from a basic workflow to an automated pipeline that validates repository changes, builds a small deployment artifact, and publishes it without manually uploading the site.

The deployed page provides a visible record of each successful run:
- Pipeline status
- GitHub Actions run number
- Short commit SHA
- Build date and time (UTC)
- Event that triggered the workflow

## What I implemented

### 1. Started with GitHub Actions

I created an initial workflow to understand how GitHub Actions executes jobs and steps. I then added a shell-based check and connected it to an automated workflow.

### 2. Wrote a shell check

The script at [tests/check.sh](tests/check.sh) uses Bash and grep to verify that README.md contains a Markdown H1 title (a line beginning with # ). It exits with a non-zero status if the requirement is not met, allowing the workflow to fail when the check fails.

### 3. Added continuous integration

The .github/workflows/ci.yml workflow:
- Runs on pushes and pull requests.
- Uses an Ubuntu runner.
- Checks out the repository with actions/checkout@v4.
- Runs bash tests/check.sh.

### 4. Built my own composite GitHub Action

I created [the build-card action](.github/actions/build-card/action.yml), a local composite action named **Build status card**.

It accepts:
- name — required name displayed on the page.
- message — optional visitor-facing message with a default value.

The action creates site/index.html and fills it with runtime information supplied by GitHub Actions, including GITHUB_RUN_NUMBER, GITHUB_SHA, and GITHUB_EVENT_NAME. It also writes a short build summary to GITHUB_STEP_SUMMARY.

This let me reuse my own action inside the deployment workflow rather than keeping all page-generation logic directly in the workflow file.

### 5. Created the CI/CD deployment workflow

The .github/workflows/pipeline.yml workflow is named pipeline and has three dependent jobs:

| Job | Responsibility |
|---|---|
| test | Checks out the code and runs the shell check. |
| build | Runs my composite action to generate the status page, then packages the site directory with actions/upload-pages-artifact@v3. |
| deploy | Publishes the uploaded artifact using actions/deploy-pages@v4. |

The workflow is configured to run:
- On pushes to main.
- On a daily schedule at 08:00 UTC (0 8 * * *).
- Manually through workflow_dispatch.

I also configured the permissions required for GitHub Pages deployment (contents: read, pages: write, and id-token: write) and a concurrency group to prevent overlapping Pages deployments.

### 6. Published the result with GitHub Pages

After the test and build jobs complete successfully, the deployment job publishes the generated site. The result is available at:

**[https://oluwaloseyiT.github.io/pipeit-weekend/](https://oluwaloseyiT.github.io/pipeit-weekend/)**

## Proof of successful pipeline runs

The following two successful deployments were captured from the live GitHub Pages status card on **7 November 2026**.

| Evidence | Run #2 | Run #3 |
|---|---|---|
| Result | All checks passed | All checks passed |
| Trigger | workflow_dispatch (manual) | push (automatic) |
| Commit shown | cade561 | bf48af5 |
| Build time shown | 13:06 UTC | 13:22 UTC |
| Deployed page | [Open live status page](https://oluwaloseyiT.github.io/pipeit-weekend/) | [Open live status page](https://oluwaloseyiT.github.io/pipeit-weekend/) |

The screenshots supplied with this project request show both status-card results. The live page is updated by subsequent deployments, so its displayed run details may change; the table above records what the screenshots captured.

### Workflow and implementation references

- [CI workflow](.github/workflows/ci.yml)
- [CI/CD deployment workflow](.github/workflows/pipeline.yml)
- [Custom composite action](.github/actions/build-card/action.yml)
- [Shell check](tests/check.sh)
- [Commit adding the CI/CD pipeline](https://github.com/oluwaloseyiT/pipeit-weekend/commit/cade5611b9c0cdf2b7932b15f1050e21077a5100)
- [Commit adding the custom action to CI](https://github.com/oluwaloseyiT/pipeit-weekend/commit/a188e3204ad0f2f2f3199419f96904d602964e3b)

## Skills demonstrated

- Continuous Integration and Continuous Deployment (CI/CD)
- GitHub Actions workflows, jobs, steps, triggers, and job dependencies
- Bash scripting and exit codes for automated checks
- Building and invoking a custom composite GitHub Action
- Passing inputs and using GitHub Actions environment variables
- Creating and packaging a static deployment artifact
- Deploying a site to GitHub Pages
- Workflow permissions and concurrency configuration
- Reading and verifying pipeline execution results

## How to run it yourself

1. Fork or clone this repository.
2. Push a change to the main branch to trigger the deployment workflow.
3. Alternatively, open Actions → pipeline → Run workflow to start it manually.
4. Open the workflow run and inspect the test, build, and deploy jobs.
5. Visit the GitHub Pages URL to see the generated status card.

## What I learned

This project helped me understand how individual automation steps fit together into a delivery pipeline: validate the change first, build only after validation succeeds, package the output, and deploy it through GitHub Pages. I also gained practical experience creating a composite action and using workflow triggers to support both automatic and manual deployments.

---

**Built by Oluwatofunmi · PipeIt Weekend · SKILL.SCH**

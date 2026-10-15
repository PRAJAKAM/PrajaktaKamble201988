# Git, GitHub, and Jenkins Integration Guide

This guide explains how Git, GitHub, and Jenkins work together in a typical CI/CD workflow.

## 1. Git, GitHub, and Jenkins

- **Git** is a distributed version control system used to track source-code changes locally.
- **GitHub** is a hosted Git platform used for remote repositories, collaboration, pull requests, code review, and access control.
- **Jenkins** is an automation server that can build, test, package, and deploy code when changes are pushed to GitHub.

A common workflow is:

```text
Developer -> git commit -> git push -> GitHub -> webhook -> Jenkins
                                                     |
                                                     v
                                           checkout, build, test, deploy
```

## 2. Install and Configure Git

Check whether Git is installed:

```bash
git --version
```

Configure the identity used in commits:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Review configuration:

```bash
git config --global --list
git config --list --show-origin
```

Useful configuration options:

```bash
git config --global init.defaultBranch main
git config --global core.autocrlf true       # Recommended on Windows
git config --global pull.rebase false       # Merge-based pull behavior
git config --global push.autoSetupRemote true
```

Use `core.autocrlf input` on Linux or macOS if the repository is shared with Windows developers.

## 3. Create or Clone a Repository

Create a new local repository:

```bash
mkdir my-project
cd my-project
git init
```

Clone an existing GitHub repository:

```bash
git clone https://github.com/OWNER/REPOSITORY.git
cd REPOSITORY
```

Clone a specific branch:

```bash
git clone --branch develop --single-branch https://github.com/OWNER/REPOSITORY.git
```

## 4. The Git Working Areas

Git changes move through three main areas:

1. **Working tree**: files currently being edited.
2. **Staging area**: changes selected for the next commit.
3. **Repository history**: committed snapshots.

Inspect the current state:

```bash
git status
git status --short
```

View unstaged changes:

```bash
git diff
```

View staged changes:

```bash
git diff --cached
```

## 5. Basic Git Commands

Stage one file:

```bash
git add path/to/file
```

Stage all changes in the repository:

```bash
git add -A
```

Commit staged changes:

```bash
git commit -m "Add login validation"
```

View recent commits:

```bash
git log --oneline --decorate --graph --all
git log -5
```

Show a commit:

```bash
git show COMMIT_ID
```

Remove a file from Git and the working tree:

```bash
git rm path/to/file
```

Rename a tracked file:

```bash
git mv old-name.md new-name.md
```

Ignore generated or local files by adding patterns to `.gitignore`:

```gitignore
# Build output
build/
target/
dist/

# Dependencies
node_modules/

# IDE and operating-system files
.vscode/
.idea/
.DS_Store
Thumbs.db

# Secrets and local configuration
.env
*.key
*.pem
```

Never commit passwords, API keys, private keys, or production credentials.

## 6. Branches

Branches allow work to be developed independently.

List branches:

```bash
git branch
git branch --all
```

Create and switch to a branch:

```bash
git switch -c feature/user-registration
```

Switch branches:

```bash
git switch main
git switch develop
```

Update the current branch from its remote branch:

```bash
git pull
```

Publish a new branch:

```bash
git push -u origin feature/user-registration
```

Delete a local branch after merging:

```bash
git branch -d feature/user-registration
```

Delete a remote branch:

```bash
git push origin --delete feature/user-registration
```

A common team convention is:

```text
main       Production-ready code
 develop   Shared integration branch
 feature/* Individual work
 hotfix/*  Urgent production fixes
```

## 7. Remotes and GitHub

A remote is a named URL for another copy of the repository. `origin` is the conventional default name.

View remotes:

```bash
git remote -v
```

Add a remote:

```bash
git remote add origin https://github.com/OWNER/REPOSITORY.git
```

Change a remote URL:

```bash
git remote set-url origin https://github.com/OWNER/REPOSITORY.git
```

Download remote references without changing local files:

```bash
git fetch origin
```

Push the current branch:

```bash
git push
```

Push a branch and set its upstream:

```bash
git push -u origin main
```

Pull remote changes:

```bash
git pull origin main
```

A safer explicit equivalent is:

```bash
git fetch origin
git merge origin/main
```

Check whether the local branch is ahead or behind:

```bash
git status -sb
git branch -vv
```

## 8. GitHub Authentication

### HTTPS

GitHub no longer accepts an account password for Git operations over HTTPS. Use a **Personal Access Token (PAT)** when prompted for a password.

Recommended approach:

1. Create a fine-grained token in GitHub Settings.
2. Grant access only to the required repository.
3. Grant the minimum permissions, such as repository contents read/write when pushing.
4. Store the token using a credential manager instead of placing it in a remote URL.

Do not use this unsafe pattern:

```bash
git remote set-url origin https://TOKEN@github.com/OWNER/REPOSITORY.git
```

### SSH

Generate an SSH key if needed:

```bash
ssh-keygen -t ed25519 -C "you@example.com"
```

Start the SSH agent and add the key. The exact command depends on the operating system and shell. Then add the contents of the public key file, normally `~/.ssh/id_ed25519.pub`, to GitHub under **Settings > SSH and GPG keys**.

Test authentication:

```bash
ssh -T git@github.com
```

Use an SSH remote:

```bash
git remote set-url origin git@github.com:OWNER/REPOSITORY.git
git push -u origin main
```

## 9. Pull Requests and GitHub Collaboration

Recommended feature workflow:

```bash
git switch main
git pull origin main
git switch -c feature/add-search
# edit files
git status
git add -A
git commit -m "Add search support"
git push -u origin feature/add-search
```

Then create a pull request in GitHub:

1. Select the feature branch as the source.
2. Select `main` or `develop` as the target.
3. Describe the change and testing performed.
4. Add reviewers.
5. Wait for required checks to pass.
6. Resolve review comments.
7. Merge using the repository's approved merge strategy.

Pull requests are useful because they provide review, automated checks, discussion, and an audit trail before code reaches a protected branch.

## 10. Merge, Rebase, and Conflict Resolution

Merge the latest `main` into the current branch:

```bash
git fetch origin
git merge origin/main
```

Rebase the current branch on the latest `main`:

```bash
git fetch origin
git rebase origin/main
```

Use rebase for local, unpublished work when your team permits it. Avoid rebasing shared branches because it rewrites commit history.

When a conflict occurs:

```bash
git status
```

Open each conflicted file and remove the conflict markers:

```text
<<<<<<< HEAD
Current branch version
=======
Incoming version
>>>>>>> origin/main
```

After resolving the file:

```bash
git add path/to/resolved-file
git commit                    # For a merge
# or
git rebase --continue        # For a rebase
```

Cancel the operation if necessary:

```bash
git merge --abort
git rebase --abort
```

## 11. Undoing Changes Safely

Discard unstaged changes in one file:

```bash
git restore path/to/file
```

Unstage a file but keep its edits:

```bash
git restore --staged path/to/file
```

Create a new commit that reverses an earlier commit:

```bash
git revert COMMIT_ID
```

`git revert` is preferred for commits already pushed to a shared branch.

Amend the most recent local commit:

```bash
git add -A
git commit --amend --no-edit
```

Use `git reset` with care, especially after pushing. Never use destructive history commands on shared branches without team approval.

## 12. Jenkins Overview

Jenkins can automatically run a pipeline when GitHub receives a push or pull request.

Typical stages are:

1. Checkout source code.
2. Install dependencies.
3. Compile or build.
4. Run unit and integration tests.
5. Publish test results and artifacts.
6. Deploy to a target environment.
7. Notify the team.

Jenkins can run jobs in several ways:

- **Freestyle project**: configured mainly through the Jenkins UI.
- **Pipeline job**: pipeline stored in Jenkins or source control.
- **Multibranch Pipeline**: automatically discovers branches and pull requests containing a `Jenkinsfile`.
- **Organization Folder**: discovers repositories across a GitHub organization.

For modern projects, a Multibranch Pipeline with a `Jenkinsfile` stored in the repository is usually the most maintainable option.

## 13. Jenkins Prerequisites

Install or make available:

- Jenkins controller and at least one build agent.
- Git on the Jenkins agent.
- The project runtime, such as Java, Node.js, Python, or Maven.
- Jenkins plugins for Pipeline, Git, GitHub integration, credentials, and the project build tool.
- Network access from Jenkins to GitHub and from GitHub to Jenkins for webhooks.

Verify tools on the Jenkins agent:

```bash
git --version
java -version
mvn --version
node --version
npm --version
```

Only install the tools required by the project.

## 14. Add GitHub Credentials to Jenkins

Do not put a GitHub token directly in a `Jenkinsfile`.

For a private repository, add credentials in Jenkins:

1. Open **Manage Jenkins > Credentials**.
2. Select the appropriate global or folder scope.
3. Add a credential.
4. Choose **Username with password** for HTTPS, where the username is the GitHub username and the password is a PAT, or choose **SSH Username with private key** for SSH.
5. Give the credential a clear ID, such as `github-read-repository`.
6. Grant only the permissions required by the job.

For checkout-only jobs, a read-only repository token is usually sufficient. Use a separate credential if Jenkins must create releases, comment on pull requests, or publish packages.

## 15. Create a Jenkins Pipeline Job

### Multibranch Pipeline

1. In Jenkins, select **New Item**.
2. Choose **Multibranch Pipeline**.
3. Add a branch source such as GitHub.
4. Select the GitHub credentials.
5. Enter the repository URL or organization.
6. Configure the scan trigger.
7. Save and run **Scan Multibranch Pipeline Now**.
8. Jenkins searches branches for a file named `Jenkinsfile`.

### Standard Pipeline Job

1. Select **New Item** and choose **Pipeline**.
2. Under **Pipeline**, choose **Pipeline script from SCM**.
3. Select Git.
4. Enter the repository URL.
5. Select the Jenkins credential.
6. Set the branch, for example `*/main`.
7. Set the script path to `Jenkinsfile`.
8. Save and build.

## 16. GitHub Webhook Configuration

A webhook tells Jenkins that a repository event occurred.

In GitHub:

1. Open the repository and select **Settings > Webhooks**.
2. Select **Add webhook**.
3. Set the Jenkins webhook URL. Common examples are:
   - `https://jenkins.example.com/github-webhook/`
   - A URL supplied by the Jenkins GitHub integration.
4. Set the content type to `application/json`.
5. Select the events required by the project, commonly push and pull request events.
6. Enable the webhook.
7. Save it and use GitHub's recent deliveries to test the connection.

In Jenkins, enable the corresponding GitHub hook trigger. For a Multibranch Pipeline, configure periodic or event-driven branch indexing according to the installed GitHub integration plugin.

For a Jenkins server running only on a developer laptop, GitHub cannot normally reach `localhost`. Use a reachable server, a secured tunnel approved by your organization, or poll GitHub temporarily while testing.

## 17. Example Jenkinsfile

Create a file named `Jenkinsfile` at the repository root:

```groovy
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        timeout(time: 30, unit: 'MINUTES')
    }

    triggers {
        // Use the GitHub webhook to start builds.
        githubPush()
    }

    environment {
        // Keep values such as URLs non-secret; use credentials() for secrets.
        CI = 'true'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                // Replace this command with the project's build command.
                sh 'mvn -B clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn -B test'
            }
            post {
                always {
                    junit testResults: 'target/surefire-reports/*.xml', allowEmptyResults: true
                }
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'Build and tests completed successfully.'
        }
        failure {
            echo 'The pipeline failed. Review the stage logs and test reports.'
        }
        always {
            cleanWs()
        }
    }
}
```

For a Windows Jenkins agent, replace `sh` with `bat`:

```groovy
stage('Build') {
    steps {
        bat 'mvn -B clean package -DskipTests'
    }
}
```

For a Node.js project, typical stages may contain:

```groovy
stage('Install') {
    steps {
        sh 'npm ci'
    }
}

stage('Test') {
    steps {
        sh 'npm test -- --ci'
    }
}

stage('Build') {
    steps {
        sh 'npm run build'
    }
}
```

Adjust commands, report paths, and artifact paths to match the project. Do not copy the example unchanged if the project uses another build system.

## 18. Credentials in a Jenkinsfile

Reference a Jenkins credential by ID instead of exposing its value:

```groovy
pipeline {
    agent any

    stages {
        stage('Publish') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'package-registry',
                        usernameVariable: 'REGISTRY_USER',
                        passwordVariable: 'REGISTRY_TOKEN'
                    )
                ]) {
                    sh 'set +x; ./publish.sh'
                }
            }
        }
    }
}
```

Important rules:

- Never echo secrets.
- Do not pass secrets as command-line arguments when the process may expose them.
- Use Jenkins credentials bindings and folder-level permissions.
- Rotate tokens periodically.
- Give tokens the smallest possible scope.
- Review logs to ensure masking is working.

## 19. Pull Request Validation

For pull request validation, the pipeline should normally:

1. Checkout the pull request revision.
2. Install dependencies from a lock file.
3. Build the application.
4. Run unit and integration tests.
5. Publish test and coverage results.
6. Report the result back to GitHub.
7. Avoid production deployment from unmerged pull requests.

A common policy is:

```text
Pull request -> build and test only
main         -> build, test, package, deploy to staging
release tag  -> build, test, deploy to production with approval
```

Use GitHub branch protection rules to require Jenkins checks before merging into `main`.

## 20. GitHub and Jenkins Integration Checklist

### Repository

- [ ] The repository has a valid `Jenkinsfile`.
- [ ] Build commands work locally.
- [ ] Test commands return a non-zero exit code on failure.
- [ ] Generated files and secrets are in `.gitignore`.
- [ ] Dependencies use a lock file where supported.

### Jenkins

- [ ] Jenkins and agents can reach GitHub.
- [ ] Git and required build tools are installed on the agent.
- [ ] Credentials are stored in Jenkins Credentials Manager.
- [ ] The job uses the correct repository and branch.
- [ ] The correct Jenkinsfile path is configured.
- [ ] Test reports and artifacts are archived.
- [ ] Workspace cleanup is enabled where appropriate.

### GitHub

- [ ] The webhook URL is reachable from GitHub.
- [ ] The webhook uses `application/json`.
- [ ] Push and pull request events are selected as needed.
- [ ] Recent webhook deliveries show successful responses.
- [ ] Branch protection requires the Jenkins check.
- [ ] The GitHub token has minimum required permissions.

## 21. Common Problems and Fixes

### `Permission denied` when pushing

Check the remote and authentication method:

```bash
git remote -v
ssh -T git@github.com
```

For HTTPS, use a valid PAT rather than an account password. For SSH, verify that the public key is registered with GitHub and that the correct remote URL is configured.

### `non-fast-forward` push rejection

Someone pushed changes that are not in the local branch:

```bash
git fetch origin
git rebase origin/main
git push
```

Resolve conflicts before continuing. Use merge instead of rebase if that is the team's agreed workflow.

### Jenkins cannot clone the repository

Check:

- Repository URL.
- Credential type and ID.
- Token permissions.
- Whether the Jenkins agent can reach GitHub.
- Whether the branch name is correct.
- Whether the SSH host key or HTTPS certificate is trusted.

### GitHub webhook does not trigger Jenkins

Check:

- Jenkins is publicly reachable from GitHub.
- The webhook URL ends with the required path, often `/github-webhook/`.
- The webhook delivery response code.
- Jenkins job trigger configuration.
- GitHub and Jenkins plugin logs.
- Whether the job is a Multibranch Pipeline that needs branch indexing.

### Jenkins says a tool is missing

Install or configure the tool on the agent, then verify the configured tool name in **Manage Jenkins > Tools**. Ensure the pipeline runs on an agent that has the tool.

### Tests pass locally but fail in Jenkins

Compare:

- Operating system and shell.
- Java, Node.js, Python, or compiler versions.
- Environment variables.
- Time zone and locale.
- File permissions.
- Network access.
- Clean checkout behavior.
- Dependency versions.

The pipeline should build from a clean workspace and should not depend on files left by a previous build.

## 22. Recommended Daily Workflow

```bash
# Start from the latest shared branch
git switch main
git pull --ff-only origin main

# Create isolated work
git switch -c feature/my-change

# Inspect, stage, and commit
git status
git diff
git add -A
git diff --cached
git commit -m "Describe the change clearly"

# Publish and open a pull request
git push -u origin feature/my-change
```

After review and merge:

```bash
git switch main
git pull --ff-only origin main
git branch -d feature/my-change
```

Use small, focused commits with clear messages. Keep commits buildable when practical so Jenkins results are meaningful and changes are easier to review or revert.

## 23. Useful Commit Message Examples

```text
Add API timeout configuration
Fix null handling in user service
Update Playwright browser setup
Refactor Jenkins test stage
Document local development setup
```

A good commit message explains the intent of the change, not just the file that changed.

## 24. Final End-to-End Example

```bash
# Clone and configure
git clone git@github.com:OWNER/REPOSITORY.git
cd REPOSITORY
git config user.name "Your Name"
git config user.email "you@example.com"

# Work on a branch
git switch -c feature/ci-pipeline
# Add or update Jenkinsfile
git add Jenkinsfile
git commit -m "Add Jenkins CI pipeline"
git push -u origin feature/ci-pipeline
```

Then:

1. Open a pull request on GitHub.
2. GitHub sends an event to Jenkins through the webhook.
3. Jenkins discovers the branch and reads `Jenkinsfile`.
4. Jenkins checks out the branch.
5. Jenkins builds and tests the code.
6. Jenkins reports the result to GitHub.
7. After approval and passing checks, merge the pull request.
8. Jenkins builds the merged `main` branch and performs the deployment stages configured for that branch.

This arrangement keeps source control, review, automation, and deployment connected while keeping credentials and build logic out of individual developer machines.

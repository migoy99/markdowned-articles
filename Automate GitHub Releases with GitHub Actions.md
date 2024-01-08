In software development, automating repetitive tasks is crucial for efficiency. GitHub Actions provides a powerful automation platform, and in this article, we'll explore a workflow that automates the process of creating GitHub releases whenever a new versioned commit or tag is pushed.

## Prerequisites
Before diving into the workflow details, make sure you have the following:
1. **A GitHub Repository:** Ensure you have a GitHub repository set up for your project.
2. **GitHub Token (Personal Access Token):** for this workflow, you'll need a GitHub token with the necessary permissions. Follow these steps to create a Personal Access Token (PAT):
    - Go to your GitHub account settings.
    - Navigate to "Developer settings" > "Personal access tokens."
    - Click on "Generate token" and provide the required scopes, including `repo` for repository access.
    - **Important:** Avoid hardcoding access tokens directly into workflow files for security reasons. Instead, store the token as a secret in your GitHub repository. To add a secret:
		- Go to your GitHub repository.
		- Navigate to "Settings" > "Secrets" > "New repository secret."
		- Enter a name for the secret (e.g., `GH_TOKEN`) and paste the token as the value.
		- Click "Add secret" to securely store the token.
1. **Basic Understanding of GitHub Actions:** Familiarize yourself with basic concepts of GitHub Actions. If you're new to GitHub Actions, the official documentation provides a great starting point.

## Set Up GitHub Actions Workflow File

```yaml
# Name of the workflow
name: create-release-on-tag-push

# Run on every commit tag which begins with "v" (e.g., "v0.1.4")
on:
  push:
    tags:
      - "v*"

# Automatically create a GitHub Release, with release details specified (the relevant commits)
jobs:
  release:
    name: "Release"
    runs-on: "ubuntu-latest"
    steps:
      - uses: "marvinpinto/action-automatic-releases@latest"
        with:
          repo_token: "${{ secrets.GH_TOKEN }}"
          prerelease: false
```

This workflow is triggered whenever a tag starting with "v" is pushed to the repository. For example, if you push a tag like "v1.0.4," the workflow will automatically create a GitHub Release.

## Automatic GitHub Release

The primary goal of this workflow is to automate the creation of GitHub releases. This is especially beneficial for versioned releases.

## Job Details

The workflow includes a job named "Release" that runs on the latest Ubuntu environment. The key step utilizes the "marvinpinto/action-automatic-releases" action.

## Automatic Release Action

The "marvinpinto/action-automatic-releases" action is responsible for automating the GitHub release creation process. Key parameters include:

- `repo_token`: GitHub token for repository access
- `prerelease`: Indicates whether the release is a pre-release or not

---

In conclusion, automating GitHub releases with GitHub Actions streamlines the versioning process, saving time and ensuring consistency. Feel free to adjust the content or let me know if you have any specific modifications in mind!
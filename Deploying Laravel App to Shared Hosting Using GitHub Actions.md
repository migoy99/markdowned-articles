
In this tutorial, I will guide you through the process of deploying a Laravel application to a shared hosting environment using GitHub Actions. We will specifically focus on deploying changes to a subdomain on a server. This tutorial assumes that you already have a Laravel application hosted on GitHub and have access to the server that will host your Laravel app.

## Setup GitHub Secrets

To maintain the security of sensitive information, we'll utilize GitHub Secrets. In your GitHub repository, navigate to `Settings` > `Secrets` and add the following secrets:

- `PROD_SSH_HOST`: The hostname or IP address of your server.
- `PROD_SSH_USER`: Your username in the server for SSH authentication.
- `PROD_SSH_PRIVATE_KEY`: Your SSH private key for authentication.
- `DEPLOYMENT_DIRECTORY`: The directory on the server where your Laravel application is hosted.
- `FETCH_CHECKOUT_TAG_SCRIPT_PATH`: The path to a script in the server that fetches and checks out the latest tag from your remote repository
- `PHP_BINARY_PATH`: This is optional. If your server has multiple PHP versions, provide the path to the specific PHP binary you want to use.

## Create a GitHub Actions Workflow File

Create a file named `.github/workflows/deploy.yml` in your Laravel project with the following content:

```yaml
name: deploy-to-subdomain
on:
  release:
    types:
      - created

jobs:
  deploy-to-server:
    name: Deploy to Server
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up SSH and Deploy changes to Server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.PROD_SSH_HOST }}
          username: ${{ secrets.PROD_SSH_USER }}
          key: ${{ secrets.PROD_SSH_PRIVATE_KEY }}

          # TAG BASED DEPLOYMENT
          # Fetches and Checkouts the latest tag from remote repository
          script: |
            cd ${{ secrets.DEPLOYMENT_DIRECTORY }}
            bash ${{ secrets.FETCH_CHECKOUT_TAG_SCRIPT_PATH }}
            composer install --no-dev
            ${{ secrets.PHP_BINARY_PATH }} artisan migrate --force
            ${{ secrets.PHP_BINARY_PATH }} artisan config:cache
            ${{ secrets.PHP_BINARY_PATH }} artisan event:cache
            ${{ secrets.PHP_BINARY_PATH }} artisan route:cache
            ${{ secrets.PHP_BINARY_PATH }} artisan view:cache
```

This GitHub Actions workflow is designed to deploy a Laravel application to a shared hosting server whenever a new release is created. The process involves checking out the latest code, setting up an SSH connection to the server, and executing deployment commands.

You might be confused about the `${{ secrets.PHP_BINARY_PATH }}` in the command. If you have multiple PHP versions installed on your server and want to specify which version your project should use, include the path to the specific PHP binary using `${{ secrets.PHP_BINARY_PATH }}`. However, if you only have one PHP version installed, you can simplify this command by using `php artisan`.  Ensure that the PHP version on your server matches the version used in your Laravel project. 

```
php artisan migrate --force
php artisan config:cache
php artisan event:cache
php artisan route:cache
php artisan view:cache
```


## Workflow Parts

```
name: deploy-to-subdomain
on:
  release:
    types:
      - created
```

- **`name`:** The name of the workflow.
- **`on`:** Specifies the trigger for the workflow. In this case, it runs when a new release is created.

```
jobs:
  deploy-to-server:
    name: Deploy to Server
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
```

- **`jobs`:** Defines a set of jobs to be executed.
- **`deploy-to-server`:** Specifies the job name.
- **`runs-on`:** Specifies the operating system for the job, in this case, Ubuntu.
- **`steps`:** Contains a list of steps to be executed within the job.

```
      - name: Set up SSH and Deploy changes to Ronins Server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.PROD_SSH_HOST }}
          username: ${{ secrets.PROD_SSH_USER }}
          key: ${{ secrets.PROD_SSH_PRIVATE_KEY }}
```

- **`appleboy/ssh-action`:** This is a GitHub Action provided by the GitHub Actions community and maintained by a user or organization named `appleboy`. This action simplifies the process of establishing an SSH connection to a remote server and executing commands.
- **`with`:** Specifies the configuration parameters for the SSH connection.

```
          # TAG BASED DEPLOYMENT
          # Fetches and Checkouts the latest tag from remote repo
          script: |
            cd ${{ secrets.DEPLOYMENT_DIRECTORY }}
            bash ${{ secrets.FETCH_CHECKOUT_TAG_SCRIPT_PATH }}
            composer install --no-dev
            ${{ secrets.PHP_BINARY_PATH }} artisan migrate --force
            ${{ secrets.PHP_BINARY_PATH }} artisan config:cache
            ${{ secrets.PHP_BINARY_PATH }} artisan event:cache
            ${{ secrets.PHP_BINARY_PATH }} artisan route:cache
            ${{ secrets.PHP_BINARY_PATH }} artisan view:cache
```

- **`cd ${{ secrets.DEPLOYMENT_DIRECTORY }}`:** Changes the working directory to the deployment directory on the server or the directory on the server where your Laravel application is hosted.
- **`bash ${{ secrets.FETCH_CHECKOUT_TAG_SCRIPT_PATH }}`:** Executes the custom script to fetch and checkout the latest tag:

```
#!/bin/bash

#Add ssh private key to ssh-agent.
eval "$(ssh-agent)" && ssh-add ~/.ssh/mtrininidad-key

#Fetch latest tags from remote repo.
git fetch --tags

#Get latest tag and store in a variable.
latestTag=$(git describe --tags `git rev-list --tags --max-count=1`)

#Checkout the latest tag.
git checkout $latestTag

#This script ensures that the SSH private key is added to the agent, fetches the #latest tags, determines the latest tag, and then checks out that tag for #deployment.
```

- **Subsequent commands:** Perform essential Laravel deployment tasks, such as installing dependencies, running migrations, and caching configurations.

## Triggering Workflow on Push to Branch

Optionally, you can trigger the workflow on each push to a specific branch. Add and modify the following section in the workflow file if you want to trigger the deployment on every push to a specific branch, in this case, production branch:

```
on:
  push:
    branches:
      - production
```


## Branch Based Deployment

Optionally, you can use branch-based deployment instead of tag-based. To perform branch-based deployment instead of tag-based, you can use the following script. Update the workflow file:

```
      # BRANCH BASED DEPLOYMENT
      script: | 
        cd ${{ secrets.DEPLOYMENT_DIRECTORY }}
        eval "$(ssh-agent)"
        ssh-add ${{ secrets.SSH_P_KEY_PATH }}
        git pull origin ${{ vars.SOURCE_BRANCH_PULL }}
        composer install --no-dev
        ${{ vars.PHP_BINARY_PATH }} artisan migrate --force
        ${{ vars.PHP_BINARY_PATH }} artisan config:cache
        ${{ vars.PHP_BINARY_PATH }} artisan event:cache
        ${{ vars.PHP_BINARY_PATH }} artisan route:cache
        ${{ vars.PHP_BINARY_PATH }} artisan view:cache

```

- **`script`:** Contains the custom script for branch-based deployment.
- **`cd ${{ secrets.DEPLOYMENT_DIRECTORY }}`:** Changes the working directory to the deployment directory on the server.
- **`eval "$(ssh-agent)"`:** Initializes the SSH agent.
- **`ssh-add ${{ secrets.SSH_P_KEY_PATH }}`:** Adds the SSH private key to the agent for authentication.
- **`git pull origin ${{ vars.SOURCE_BRANCH_PULL }}`:** Pulls the latest changes from the specified source branch.
- **Subsequent commands**: Perform essential Laravel deployment tasks, such as installing dependencies, running migrations, and caching configurations.


##  To wrap things up

Congratulations! You've successfully set up a GitHub Actions workflow for deploying your Laravel application to a server. Whether you prefer tag-based deployment for releases or branch-based deployment, this workflow accommodates both scenarios.

Remember, this tutorial provides a foundational structure, and you have the flexibility to customize the workflow based on your specific needs. Feel free to tweak the deployment scripts, adjust triggers, or incorporate additional steps to align with your project requirements.

# GitHub Actions for Next.js Build and Deployment

This repository contains a setup for automating the build and deployment of a Next.js application using GitHub Actions. The provided workflow will be triggered on every push to the repository and will deploy the latest changes to a remote server via SSH.

## Prerequisites

Before setting up the GitHub Actions workflow, ensure you have the following:

- **Remote Server:** A server where the Next.js application will be deployed.
- **SSH Access:** Ensure you have SSH access to the server and have created the necessary credentials.
- **PM2:** The server should have PM2 installed to manage the Node.js application process.

### Secrets Configuration

You need to configure the following secrets in your GitHub repository:

- `SERVER_HOST`: The hostname or IP address of the remote server.
- `SERVER_USERNAME`: The SSH username for accessing the server.
- `SERVER_PASSWORD`: The SSH password for the username.
- `SERVER_PORT`: The SSH port (default is 22).
- `SERVER_PATH`: The path on the server where the repository is cloned and the Next.js application is deployed.

To add these secrets:

- Go to your GitHub repository.
- Click on Settings.
- Click on Secrets and variables > Actions.
- Click New repository secret and add each of the above secrets.

![GitHub Actions for Next.js Build and Deployment](https://github.com/user-attachments/assets/27a5f65a-e4b2-4835-ab90-07b789026ab7)

## Workflow Setup

The deploy.yml file defines the workflow for building and deploying the Next.js application. This file should be placed in the .github/workflows directory of your repository.

### File Structure

The deploy.yml file should be located at the following path in your repository:

```markdown
.github/
└── workflows/
    └── deploy.yml
```

## Workflow Configuration

Here is the deploy.yml file you need to set up in your repository:

```yaml
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Run and deploy
        uses: appleboy/ssh-action@v0.1.9
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USERNAME }}
          password: ${{ secrets.SERVER_PASSWORD }}
          port: ${{ secrets.SERVER_PORT }}
          script: |
            cd ${{ secrets.SERVER_PATH }}
            git pull
            npm install --force
            npm run build
            if [ -f "package.json" ]; then
              if pm2 describe "nextjs-website" > /dev/null; then
                pm2 restart "nextjs-website"
              else
                pm2 start --name "nextjs-website" "npm run start -- -p 3001"
              fi
              pm2 save
            else
              echo "package.json not found. Skipping start/restart."
            fi
```

### How It Works

- **Trigger:** The workflow is triggered on every push to the repository.
- **Checkout: The repository code is checked out using the actions/checkout action.
- **Deploy: The workflow connects to the remote server via SSH using the appleboy/ssh-action action. It navigates to the specified server path, pulls the latest changes, installs dependencies, builds the Next.js application, and manages the application process using PM2.

### Key Steps

- **Git Pull: Updates the repository on the server with the latest code.
- **Install Dependencies: Installs the required Node.js dependencies, forcing installation to ensure compatibility.
- **Build: Builds the Next.js application.
- **Manage with PM2:
If the package.json file exists:
If PM2 is already running the application (nextjs-website), it will be restarted.
If not, PM2 will start the application on port 3001.
PM2 saves the current process list to be restarted on system reboot.

### Conclusion

This GitHub Actions workflow automates the process of deploying a Next.js application to a remote server. By configuring the necessary secrets and setting up the deploy.yml file, you can ensure that your application is consistently and reliably deployed with every push to the repository.

Copyright 2024, Max Base

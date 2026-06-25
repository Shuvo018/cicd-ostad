# CI/CD Pipeline Documentation

This document explains the Continuous Integration and Continuous Deployment (CI/CD) pipeline implemented in this project.

# Understand basic CICD
```bash

name: CICD pipeline

on:
    push:
        branches:
            - main
    pull_request:
        branches:
            - main

jobs:
    print-hello:
        runs-on: ubuntu-latest

        steps:
            - name: Checkout code
              uses: actions/checkout@v3 # This action checks-out your repository under $GITHUB_WORKSPACE, so your workflow can access it.

              # Just print Hello, world
            - name: Print Hello
              run: echo "Hello, World!"
            
            # Open file and read the file
            - name: Open hello
              run: cat hello.txt
    

```

# EC2 Deploy CICD
```bash

jobs:
    Deploy:
        runs-on: self-hosted

        steps:
            - name: Checkout the home directory
              run: |
                cd /home/ubuntu/cicd-ostad
                git pull
                pm2 status
                pm2 restart all
                

```

# Test - 

setup environment like local environment

```bash

    Test:
        runs-on: ubuntu-latest

        steps:
            - name: Git clone
              uses: actions/checkout@v2

            - name: install the node
              uses: actions/setup-node@v2
              with:
                node-version: '22'

            - name: install packages
              run: npm install

            - name: Run the test
              run: npm run test



```


## Prerequisites

Before setting up the CI/CD pipeline, ensure the following prerequisites are met on your self-hosted runner:

### 1. EC2 Configuration PM2
```bash
sudo apt update
sudo apt install curl wget gnupg

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash

nvm --version

nvm install node

node -v

npm install

npm start

# Install PM2 globally on the runner
npm install -g pm2

# Verify PM2 installation
pm2 --version


# go to repo settings -> Actions -> Runners
# create new self-hosted runner
# Follow github instractions

# last instead of
# Run it!
$ ./run.cmd

# this commend you may use below commend to run application after exit the ec2 terminal
 
sudo ./svc.sh install
sudo ./svc.sh start

```



## Overview

The pipeline consists of two main jobs:
1. Testing
2. Build and Deployment

## Workflow Triggers

The pipeline is triggered on:
- Push to `main` branch
- Pull requests to `main` branch

## Jobs Details

### 1. Test Job

**Environment:**
- Runs on a self-hosted runner
- Uses Node.js version 20

**Steps:**
1. **Code Checkout**: Uses `actions/checkout@v4` to get the latest code
2. **Node.js Setup**: Configures Node.js environment using `actions/setup-node@v4`
3. **Dependencies**: Installs project dependencies using `npm install`
4. **Testing**: Runs the test suite using `npm test`

### 2. Build and Deploy Job

**Dependencies:**
- Only runs after the test job successfully completes
- Runs on a self-hosted runner

**Steps:**
1. **Code Checkout**: Fresh checkout of the code
2. **Build**: 
   - Installs dependencies
   - Prepares the application for deployment
3. **Deployment**:
   - Uses PM2 for process management
   - Handles graceful shutdown of existing processes
   - Starts the application with proper path configuration
   - Persists PM2 process list

## Process Management

### PM2 Process Details
- **Process Name**: `node-app`
- **Start Command**: `pm2 start src/server.js --name node-app`
- **Process List**: Saved automatically after deployment
- **Logs Location**: `~/.pm2/logs/`
  - Application logs: `node-app-out.log`
  - Error logs: `node-app-error.log`

### PM2 Common Commands
```bash
# List all processes
pm2 list

# Monitor processes
pm2 monit

# View logs
pm2 logs node-app

# Restart application
pm2 restart node-app

# Stop application
pm2 stop node-app

# Delete application
pm2 delete node-app
```

### Deployment Process
1. **Cleanup**: Removes existing process if running
   ```bash
   pm2 delete node-app || true
   ```

2. **Start Application**: Launches with absolute path
   ```bash
   pm2 start "${WORKSPACE_PATH}/src/server.js" --name node-app
   ```

3. **Save Process List**: Persists PM2 configuration
   ```bash
   pm2 save
   ```

## Error Handling

- Uses `|| true` with PM2 delete command to handle cases where the process doesn't exist
- Ensures proper workspace path resolution using environment variables
- PM2 automatically handles process crashes and restarts

## Security Considerations

- Runs on self-hosted runner for better control and security
- Uses explicit Node.js version for consistency
- Maintains separate jobs for testing and deployment
- PM2 runs with limited user permissions

## Maintenance

To maintain this pipeline:
1. Keep Node.js version updated
2. Monitor PM2 logs for any issues
3. Regularly update GitHub Actions dependencies
4. Review and update test coverage as needed
5. Periodically check PM2 process status and logs
6. Clean up PM2 logs to prevent disk space issues
7. Update PM2 to latest version when needed:
   ```bash
   npm install -g pm2@latest
   ```

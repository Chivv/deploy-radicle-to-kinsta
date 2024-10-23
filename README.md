# Kinsta Deployments with GitHub Actions

This repository contains a GitHub Actions workflow for automating deployments to different environments on Kinsta. The workflow is designed to deploy to both `development` and `production` environments based on the branch you push to.

## Features

- **Automated Deployment**: Automatically deploys to the appropriate environment (`development` or `production`) based on the branch you push to.
- **Environment-Specific Configuration**: Uses GitHub's `matrix` feature to handle different configurations for `development` and `production`.
- **Secure SSH Setup**: Securely sets up SSH keys and known hosts for the remote server.
- **Composer & NPM Integration**: Installs PHP dependencies using Composer and builds JavaScript assets using NPM.
- **Rsync Deployment**: Efficiently syncs files to the remote server using `rsync`.
- **Symlink Management**: Automatically handles environment files and public directories with symlinks.
- **Automatic Cache Clearing**: Clears the Kinsta cache after deployment.
- **Old Deployment Cleanup**: Cleans up older deployments, keeping only the latest 3 releases.

## Workflow Overview

The deployment workflow is triggered in two ways:

1. **Pushing to the `development` branch** triggers a deployment to the `development` environment.
2. **Pushing to the `master` branch** triggers a deployment to the `production` environment.

The workflow uses a **matrix strategy** to deploy based on the branch and dynamically sets environment-specific variables such as SSH credentials, remote directory paths, and Kinsta domain URLs.

### Key Steps in the Workflow

- **Checkout Code**: The repository code is checked out using the [checkout action](https://github.com/actions/checkout).
- **Node.js & PHP Setup**: Node.js is set up for building JavaScript assets, and PHP is set up for Composer dependencies using the [setup-node](https://github.com/actions/setup-node) and [setup-php](https://github.com/shivammathur/setup-php) actions.
- **SSH Configuration**: SSH keys and known hosts are configured for secure communication with the remote server.
- **Composer & NPM Dependencies**: PHP and JavaScript dependencies are installed and built.
- **Rsync Deployment**: Files are deployed to the remote server using `rsync`, excluding unnecessary files like `.git`, `.env`, and others.
- **Symlink & Directory Management**: The workflow ensures correct symlinking for the `.env` file and shared uploads directory. The current release is promoted by creating a `current` symlink to the new deployment.
- **Cache Clearing**: After deployment, the Kinsta cache is cleared via an HTTP request.
- **Deployment Cleanup**: Older deployments are removed, keeping only the latest 3 releases.

### Shared Directory Requirements

The following directories and files need to exist on your Kinsta server:

1. **Uploads Directory**: The shared uploads directory should be located at `[Kinsta_Root_Path]/shared/uploads`. This path is symlinked during the deployment to ensure all uploaded media files are preserved across deployments.

2. **Environment File (`.env`)**: The shared `.env` file should be located at `[Kinsta_Root_Path]/shared/.env`. This ensures that environment-specific variables are consistently applied across deployments.

## Setup Instructions

### 1. Secrets Configuration

To use this workflow, you need to set up the following secrets in your repository:

- **SSH_KEY**: The private SSH key used to connect to the server.
- **DEVELOPMENT_HOST**: The SSH host for the development server.
- **DEVELOPMENT_PORT**: The SSH port for the development server.
- **DEVELOPMENT_SSH_USERNAME**: The SSH username for the development server.
- **DEVELOPMENT_REMOTE_DIR**: The remote directory path for deployments on the development server.
- **DEVELOPMENT_DOMAIN**: The Kinsta domain for the development server (for cache clearing).
- **PRODUCTION_HOST**: The SSH host for the production server.
- **PRODUCTION_PORT**: The SSH port for the production server.
- **PRODUCTION_SSH_USERNAME**: The SSH username for the production server.
- **PRODUCTION_REMOTE_DIR**: The remote directory path for deployments on the production server.
- **PRODUCTION_DOMAIN**: The Kinsta domain for the production server (for cache clearing).

### 2. Customizing the Workflow

You can customize the workflow by adjusting the matrix in `.github/workflows/deploy.yaml` to include additional environments (e.g., `staging`), or modify the steps to suit your project's needs.

### 3. Manually Triggering the Workflow

In addition to automatically triggering on pushes to `development` or `master`, the workflow can also be triggered manually via the GitHub Actions UI. You can use the `workflow_dispatch` event to deploy at any time.

## Usage

### Pushing to `development` Branch

When you push code to the `development` branch, the workflow will automatically deploy to the `development` environment, using the settings specified in the matrix.

### Pushing to `master` Branch

When you push code to the `master` branch, the workflow will automatically deploy to the `production` environment, using the production-specific configuration.

## Example Workflow

Here’s an example of how the matrix is used to define branch-specific environments in the workflow:

```yaml
strategy:
  matrix:
    environment: [development, production]
    include:
      - environment: development
        branch: development
        host: ${{ secrets.DEVELOPMENT_HOST }}
        port: ${{ secrets.DEVELOPMENT_PORT }}
        user: ${{ secrets.DEVELOPMENT_SSH_USERNAME }}
        remote_dir: ${{ secrets.DEVELOPMENT_REMOTE_DIR }}
        domain: ${{ secrets.DEVELOPMENT_DOMAIN }}
      - environment: production
        branch: master
        host: ${{ secrets.PRODUCTION_HOST }}
        port: ${{ secrets.PRODUCTION_PORT }}
        user: ${{ secrets.PRODUCTION_SSH_USERNAME }}
        remote_dir: ${{ secrets.PRODUCTION_REMOTE_DIR }}
        domain: ${{ secrets.PRODUCTION_DOMAIN }}

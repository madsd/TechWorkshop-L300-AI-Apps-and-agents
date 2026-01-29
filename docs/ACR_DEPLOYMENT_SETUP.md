# Azure Container Registry Deployment Setup

This document explains how to set up the GitHub Actions workflow for deploying the application to Azure Container Registry (ACR).

## Overview

The workflow automatically builds a Docker image from the `src/` folder and pushes it to Azure Container Registry whenever code is pushed to the `main` branch.

## Prerequisites

1. An Azure Container Registry (ACR) instance
2. Access to configure GitHub repository secrets

## Required GitHub Secrets

You need to configure the following secrets in your GitHub repository (Settings → Secrets and variables → Actions):

### ACR Authentication Secrets

| Secret Name | Description | Example |
|------------|-------------|---------|
| `ACR_LOGIN_SERVER` | The login server URL for your Azure Container Registry | `myregistry.azurecr.io` |
| `ACR_USERNAME` | The username for ACR authentication | Your ACR username |
| `ACR_PASSWORD` | The password for ACR authentication | Your ACR password |
| `ACR_NAME` | The name of your Azure Container Registry | `myregistry` |

### Application Configuration Secret

| Secret Name | Description | Format |
|------------|-------------|--------|
| `ENV` | The contents of the .env file to be injected into the Docker image | Multi-line text with KEY=value pairs |

#### ENV Secret Format

The `ENV` secret should contain all environment variables needed by the application, one per line:

```
OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT=true
AZURE_TRACING_GEN_AI_CONTENT_RECORDING_ENABLED=true

# Microsoft Foundry credentials
FOUNDRY_ENDPOINT=https://your-endpoint.azure.com
FOUNDRY_KEY=your-key-here
FOUNDRY_API_VERSION=2025-01-01-preview

# GPT credentials
gpt_endpoint=https://your-gpt-endpoint.azure.com
gpt_deployment=gpt-5-mini
gpt_api_key=your-api-key
gpt_api_version=2025-01-01-preview

# ... (add all your environment variables)
```

You can use `src/env_sample.txt` as a template for the structure.

## How to Set Up Secrets

1. Go to your GitHub repository
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Add each secret with its corresponding name and value
5. For the `ENV` secret:
   - Copy your complete environment configuration
   - Paste it as a multi-line text value
   - **Important:** Do NOT commit your actual .env file to the repository

## How the Workflow Works

1. **Trigger:** The workflow runs automatically when code is pushed to the `main` branch
2. **Checkout:** The workflow checks out the repository code
3. **Login:** Authenticates with Azure Container Registry using the provided credentials
4. **Build:** 
   - Builds the Docker image from the `src/` folder
   - Injects the `ENV` secret content into the image as a `.env` file during build time
   - Uses BuildKit for efficient caching
5. **Tag:** Tags the image with both the commit SHA and `latest`
6. **Push:** Pushes the image to your Azure Container Registry

## Security Notes

- The `.env` file is listed in `.gitignore` and will never be committed to the repository
- Environment variables are injected at build time using Docker build arguments
- All secrets are encrypted and stored securely in GitHub
- The ENV content is written to the .env file inside the Docker image, not in the repository

## Workflow File Location

The workflow is defined in: `.github/workflows/deploy-to-acr.yml`

## Image Tags

The workflow creates two tags for each build:
- `<ACR_LOGIN_SERVER>/techworkshop-ai-app:<commit-sha>` - Specific version
- `<ACR_LOGIN_SERVER>/techworkshop-ai-app:latest` - Always points to the most recent build

## Troubleshooting

### Workflow fails with authentication error
- Verify that `ACR_LOGIN_SERVER`, `ACR_USERNAME`, and `ACR_PASSWORD` are correctly set
- Check that your ACR credentials are still valid

### Application fails to start due to missing environment variables
- Verify that all required variables are included in the `ENV` secret
- Check that the format matches the expected KEY=value format
- Review the `env_sample.txt` file for the complete list of required variables

### Build fails
- Check the workflow logs in the Actions tab
- Verify that the Dockerfile in `src/` is valid
- Ensure all dependencies in `requirements.txt` are available

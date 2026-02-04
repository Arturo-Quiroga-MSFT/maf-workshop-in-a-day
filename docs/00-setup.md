# 00: Development Environment Setup

This session covers setting up the development environment for the workshop.

## Prerequisites

- A Chromium-based web browser ([Microsoft Edge](https://microsoft.com/edge), [Google Chrome](http://chrome.google.com), etc.)
- [Azure subscription](https://azure.microsoft.com/free)
- [GitHub personal account (free)](http://github.com/signup) 👉 If using a work account, it may malfunction according to company policies, so please use a personal account.
- [Microsoft Copilot Studio trial subscription](https://go.microsoft.com/fwlink/?LinkId=2107702)

## Opening GitHub Codespaces

This workshop uses [GitHub Codespaces](https://docs.github.com/codespaces) to maintain a consistent development environment.

1. Click the button below to create a new GitHub Codespaces instance.

   [![GitHub Codespaces 인스턴스 생성하기](https://github.com/codespaces/badge.svg)](https://codespaces.new/Azure-Samples/maf-workshop-in-a-day-ko)

1. Once the GitHub Codespaces instance is created, run the following commands one by one in the terminal to verify that the necessary environment has been set up correctly.

    ```bash
    # .NET SDK
    dotnet --list-sdks

    # node.js
    node --version
    npm --version

    # PowerShell
    pwsh --version

    # Docker
    docker info

    # azd CLI
    azd version

    # az CLI
    az --version
    az bicep version

    # Aspire CLI
    aspire --version
    ```

1. Check the GitHub repository status.

    ```bash
    git remote -v
    ```

   You should see output similar to:

    ```text
    origin  https://github.com/Azure-Samples/maf-workshop-in-a-day-ko.git (fetch)
    origin  https://github.com/Azure-Samples/maf-workshop-in-a-day-ko.git (push)
    ```

   If the output doesn't match the above, delete the GitHub Codespaces instance and recreate it.

1. Run the following command to fork the repository to your account using the GitHub Codespaces instance.

    ```bash
    git remote -v > remote.txt
    git add . && git commit -m "Add remote.txt for forking"
    ```

   You will probably see a message similar to:

    ```text
    You don't have write access to the Azure-Samples/maf-workshop-in-a-day-ko repository, so you cannot push changes to it.
    To obtain write access we will point this codespace at your fork of Azure-Samples/maf-workshop-in-a-day-ko, creating that fork if it doesn't exist.
    
    Would you like to proceed?
    ```

   Press `y` to continue. This will automatically fork the current repository to your account.

1. Check the repository status again.

    ```bash
    git remote -v
    ```

   This time you should see:

    ```text
    origin  https://github.com/<YOUR_GITHUB_ID>/maf-workshop-in-a-day-ko.git (fetch)
    origin  https://github.com/<YOUR_GITHUB_ID>/maf-workshop-in-a-day-ko.git (push)
    upstream        https://github.com/Azure-Samples/maf-workshop-in-a-day-ko (fetch)
    upstream        https://github.com/Azure-Samples/maf-workshop-in-a-day-ko (push)
    ```

   If the output doesn't match, recreate the GitHub Codespaces instance and repeat this process.

## Setting Repository Root

1. Run the following command to set the `$REPOSITORY_ROOT` environment variable.

    ```bash
    # zsh/bash
    REPOSITORY_ROOT=$(git rev-parse --show-toplevel)
    ```

    ```powershell
    # PowerShell
    $REPOSITORY_ROOT = git rev-parse --show-toplevel
    ```

## GitHub Models Setup

> **NOTE**: If you cannot use an Azure subscription, you can use the [gpt-5-mini](https://github.com/marketplace/models/azure-openai/gpt-5-mini) model provided by [GitHub Models](https://docs.github.com/github-models) for free.

1. Create a [Personal Access Token (PAT)](https://docs.github.com/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens). Note that without the `models:read` permission, you won't be able to access GitHub Models.

1. After creating the PAT, keep it safe. Once generated, tokens cannot be viewed again, so if lost, you'll need to create a new one.

    ```bash
    # zsh/bash
    githubToken="{{GITHUB_PAT}}"
    ```

    ```powershell
    # PowerShell
    $githubToken = "{{GITHUB_PAT}}"
    ```

## Azure Login

> **NOTE**: Proceed only if you have been provided with an Azure subscription. Depending on the workshop, an Azure subscription may not be provided.

1. Run the following commands to log in to Azure cloud.

    ```bash
    # Azure Developer CLI 로그인
    azd auth login --use-device-code=true

    # Azure CLI 로그인
    az login --use-device-code
    ```

1. After logging in, run the following commands to verify successful login.

    ```bash
    # Azure Developer CLI 로그인 확인
    azd auth login --check-status
    
    # Azure CLI 로그인 확인
    az account show
    ```

## Creating Azure OpenAI Instance

> **NOTE**: Proceed only if you have been provided with an Azure subscription. Depending on the workshop, an Azure subscription may not be provided.

1. Verify that you are in the repository root directory.

    ```bash
    cd $REPOSITORY_ROOT
    ```

1. Run the following command to create an Azure OpenAI instance.

    ```bash
    azd up
    ```

   When prompted with the following questions, provide appropriate answers:

   - `? Enter a unique environment name:` 👉 Environment name (e.g., `mafworkshop-2026`)
   - `? Enter a value for the 'location' infrastructure parameter:` 👉 Select region (e.g., `Australia East`)

   After waiting a moment, you can confirm that the Azure OpenAI instance has been created.

   > In some cases, you may need to set the `AZURE_TENANT_ID` environment variable.
   >
   > ```bash
   > # zsh/bash
   > export AZURE_TENANT_ID=$(az account show --query "tenantId" -o tsv)
   > ```
   >
   > ```powershell
   > # PowerShell
   > $env:AZURE_TENANT_ID = az account show --query "tenantId" -o tsv
   > ```

1. Run the following commands to check the Azure OpenAI instance endpoint and API key.

    ```bash
    # zsh/bash
    endpoint=$(azd env get-value 'AZURE_OPENAI_ENDPOINT')
    apiKey=$(az cognitiveservices account keys list --name $(azd env get-value 'AZURE_OPENAI_NAME') --resource-group rg-$(azd env get-value 'AZURE_ENV_NAME') --query "key1" -o tsv)
    ```

    ```powershell
    # PowerShell
    $endpoint = azd env get-value 'AZURE_OPENAI_ENDPOINT'
    $apiKey = az cognitiveservices account keys list --name $(azd env get-value 'AZURE_OPENAI_NAME') --resource-group rg-$(azd env get-value 'AZURE_ENV_NAME') --query "key1" -o tsv
    ```

---

Congratulations! You have completed the basic development environment setup for the workshop. Now proceed to the next step!

👈 [README](../README.md) | [01: Building a Single Agent with Microsoft Agent Framework](./01-single-agent-with-maf.md) 👉

# 07: Developing an Agent in Copilot Studio and Integrating MCP Server (Optional)

In this session, we will integrate the [MCP server](./05-mcp-server-development.md) we created earlier with an agent in [Copilot Studio](https://learn.microsoft.com/microsoft-copilot-studio/fundamentals-what-is-copilot-studio).

## Session Goals

- You will be able to integrate a local MCP server with Copilot Studio.
- You will be able to integrate a remote MCP server with Copilot Studio.

## Architecture

After completing this session, the following system will be created.

![Session Architecture](./images/step-07-architecture.png)

## Prerequisites

It is assumed that you have completed all the development environment setup in the previous [00: Development Environment Setup](./00-setup.md).

## Repository Root Setup

1. Run the following command to set the `$REPOSITORY_ROOT` environment variable.

    ```bash
    # zsh/bash
    REPOSITORY_ROOT=$(git rev-parse --show-toplevel)
    ```

    ```powershell
    # PowerShell
    $REPOSITORY_ROOT = git rev-parse --show-toplevel
    ```

## Copying the Starting Project

We have prepared the starting project needed for this workshop. The project structure of the starting project is as follows.

```text
save-points/
└── step-07/
    └── start/
        ├── infra/
        │   └── < bicep files >
        ├── MafWorkshop.sln
        └── MafWorkshop.McpTodo/
            ├── Properties/
            │   └── launchSettings.json
            ├── TodoDbContext.cs
            ├── Program.cs
            ├── appsettings.json
            └── MafWorkshop.McpTodo.csproj
```

> Project Introduction:
>
> - `infra`: Bicep file directory for Azure cloud resource deployment
> - `MafWorkshop.McpTodo`: MCP server project for to-do list management

1. If you have a `workshop` directory from the previous exercise, delete it or rename it. Example: `workshop-step-06`
1. Open a terminal and run the following commands in order to create the exercise directory and copy the starting project.

    ```bash
    # zsh/bash
    rm -rf $REPOSITORY_ROOT/workshop && \
        mkdir -p $REPOSITORY_ROOT/workshop && \
        cp -a $REPOSITORY_ROOT/save-points/step-07/start/. $REPOSITORY_ROOT/workshop/
    ```

    ```powershell
    # PowerShell
    Remove-Item -Path $REPOSITORY_ROOT/workshop -Recurse -Force && `
        New-Item -Type Directory -Path $REPOSITORY_ROOT/workshop -Force && `
        Copy-Item -Path $REPOSITORY_ROOT/save-points/step-07/start/* -Destination $REPOSITORY_ROOT/workshop -Recurse -Force
    ```

## Building and Running the Starting Project

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Build the entire project.

    ```bash
    dotnet restore && dotnet build
    ```

## Accessing Copilot Studio and Creating an Agent

> Use the Copilot Studio access information provided by the workshop facilitator.
>
> **Important**: The Copilot Studio UI may change over time, so screenshots may differ.

1. Log in to [Copilot Studio](https://copilotstudio.microsoft.com).

   ![Copilot Studio - Home Screen](./images/step-07-image-06.png)

1. Go to the Agent tab and click the [➕ Create blank agent] button.

   ![Copilot Studio - Agent Tab](./images/step-07-image-07.png)

   An agent has been created.

   ![Copilot Studio - Agent Creation Result](./images/step-07-image-08.png)

## Running the Local MCP Server

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Run the MCP server application.

    ```bash
    dotnet run --project ./MafWorkshop.McpTodo
    ```

   ![Local MCP Server Run](./images/step-07-image-01.png)

1. The MCP server is currently only running locally, so it cannot be accessed from outside. Open the port so it can be accessed externally as shown in the figure below. The MCP server currently uses port `5497`.

   ![Local MCP Server Run - Port Forwarding](./images/step-07-image-02.png)

   Adjust the port to `Public` so anyone can access it.

   ![Local MCP Server Run - Port Forwarding - Public Port Setting](./images/step-07-image-03.png)

   The MCP server port is now publicly accessible via the internet.

   ![Local MCP Server Run - Port Forwarding - Port Public](./images/step-07-image-05.png)

   Copy the access URL value from the [Forwarded Address] column. The URL format is similar to `https://{instancename}-{randomchars}-{portnumber}.app.github.dev/`. For example, let's say it's `https://laughing-trout-jj69pprrgwcj6px-5497.app.github.dev/`.

## Connecting and Running Local MCP Server with Copilot Studio Agent

1. Go to the [Tools] tab at the top of the agent you created earlier and click the [➕ Add a tool] button.

   ![Copilot Studio - Create New Tool](./images/step-07-image-09.png)

   Then click the [➕ New tool] button.

   ![Copilot Studio - New Tool Button](./images/step-07-image-10.png)

1. Click the [Model Context Protocol] button.

   ![Copilot Studio - New MCP Server Button](./images/step-07-image-11.png)

1. Enter the MCP server connection information as shown in the figure below. Then click the [Create] button.

   ![Copilot Studio - Agent - MCP Server Information Input](./images/step-07-image-12.png)

   - `Server name`: Enter `Todo Manager Local XXX` 👈. XXX is a random number or characters
   - `Server description`: Enter `MCP server responsible for creating/updating/deleting to-do list items.` 👈
   - `Server URL`: Enter the public address for the local MCP server you copied earlier + `/mcp` (e.g., `https://laughing-trout-jj69pprrgwcj6px-5497.app.github.dev/mcp`)
   - `Authentication`: Select `None` 👈

1. When the following screen appears, click the [Not connected] button and then click the [Create new connection] button.

   ![Copilot Studio - Agent - MCP Server Connection Creation Request](./images/step-07-image-13.png)

   Then click the [Create] button to connect to the local MCP server.

   ![Copilot Studio - Agent - MCP Server Connection Creation](./images/step-07-image-14.png)

   Then click the [Add and configure] button to add the local MCP server to the agent.

1. In Copilot Studio, go to the [Tools] tab and click the [➕ New tool] button.

   ![Copilot Studio - Create New Tool](./images/step-07-image-15.png)

   You can see the Tools defined in the MCP server.

   ![Copilot Studio - MCP Server Tool List](./images/step-07-image-16.png)

1. Click the [Settings] button.

   ![Copilot Studio - Agent Settings](./images/step-07-image-17.png)

   You can see the MCP server connected to the current agent. Click the [Connect] link.

   ![Copilot Studio - Agent Settings - MCP Server Connection](./images/step-07-image-18.png)

   Click the [Submit] button.

   ![Copilot Studio - Agent Settings - MCP Server Connection Connect](./images/step-07-image-19.png)

   Connection is complete. Click the [X] button in the upper right to exit the agent Settings screen.

   ![Copilot Studio - Agent Settings - MCP Server Connection Complete](./images/step-07-image-20.png)

1. Try running various prompts in the test session on the right and check the results as shown below.

   ![Copilot Studio - Agent Execution Result #1](./images/step-07-image-21.png)

   ![Copilot Studio - Agent Execution Result #2](./images/step-07-image-22.png)

   ![Copilot Studio - Agent Execution Result #3](./images/step-07-image-23.png)

## Stopping the Local MCP Server

1. Press `CTRL`+`C` in the terminal to stop the application.

## Deploying the Remote MCP Server

> **NOTE**: Proceed if you have been provided with an Azure subscription. Depending on the workshop, an Azure subscription may not be provided.

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Run the following command to deploy the MCP server.

    ```bash
    azd up
    ```

   When the following questions appear, enter appropriate values.

   - `? Enter a unique environment name:` 👉 Environment name (e.g., `mafworkshop-2026`)
   - `? Enter a value for the 'location' infrastructure parameter:` 👉 Select region (e.g., `Korea Central`)

   Wait a moment and you can verify that the Azure Container Apps instance for the MCP server has been created.

1. Run the following command to get the URL value of the Azure Container Apps instance. The URL format is `mafworkshop-mcptodo.{randomstring}-{randomnumber}.{region}.azurecontainerapps.io`.

    ```bash
    azd env get-value AZURE_RESOURCE_MAFWORKSHOP_MCPTODO_FQDN
    ```

## Connecting and Running Remote MCP Server with Copilot Studio

> **NOTE**: Proceed if you have been provided with an Azure subscription. Depending on the workshop, an Azure subscription may not be provided.

1. Go to the [Tools] tab at the top of the agent you created earlier and click the [➕ Add a tool] button.

   ![Copilot Studio - Create New Tool](./images/step-07-image-09.png)

   Then click the [➕ New tool] button.

   ![Copilot Studio - New Tool Button](./images/step-07-image-10.png)

1. Click the [Model Context Protocol] button.

   ![Copilot Studio - New MCP Server Button](./images/step-07-image-11.png)

1. Enter the MCP server connection information as shown in the figure below. Then click the [Create] button.

   ![Copilot Studio - Agent - MCP Server Information Input](./images/step-07-image-24.png)

   - `Server name`: Enter `Todo Manager Remote XXX` 👈. XXX is a random number or characters
   - `Server description`: Enter `MCP server responsible for creating/updating/deleting to-do list items.` 👈
   - `Server URL`: Enter the public address for the remote MCP server you copied earlier + `/mcp` (e.g., `mafworkshop-mcptodo.{randomstring}-{randomnumber}.{region}.azurecontainerapps.io/mcp`)
   - `Authentication`: Select `None` 👈

1. When the following screen appears, click the [Not connected] button and then click the [Create new connection] button.

   ![Copilot Studio - Agent - MCP Server Connection Creation Request](./images/step-07-image-25.png)

   Then click the [Create] button to connect to the remote MCP server.

   ![Copilot Studio - Agent - MCP Server Connection Creation](./images/step-07-image-26.png)

   Then click the [Add and configure] button to add the remote MCP server to the agent.

1. In Copilot Studio, go to the [Tools] tab and click the [➕ New tool] button.

   ![Copilot Studio - Create New Tool](./images/step-07-image-27.png)

   You can see the Tools defined in the MCP server.

   ![Copilot Studio - MCP Server Tool List](./images/step-07-image-28.png)

1. Click the [Settings] button.

   ![Copilot Studio - Agent Settings](./images/step-07-image-29.png)

   You can see the MCP server connected to the current agent. Click the [Connect] link.

   ![Copilot Studio - Agent Settings - MCP Server Connection](./images/step-07-image-30.png)

   Click the [Submit] button.

   ![Copilot Studio - Agent Settings - MCP Server Connection Connect](./images/step-07-image-31.png)

   Connection is complete. Click the [X] button in the upper right to exit the agent Settings screen.

   ![Copilot Studio - Agent Settings - MCP Server Connection Complete](./images/step-07-image-32.png)

1. Try running various prompts in the test session on the right and check the results as shown below.

   ![Copilot Studio - Agent Execution Result #1](./images/step-07-image-33.png)

   ![Copilot Studio - Agent Execution Result #2](./images/step-07-image-34.png)

   ![Copilot Studio - Agent Execution Result #3](./images/step-07-image-35.png)

## Deleting Remote MCP Server Resources

1. Run the following command to delete the deployed MCP server application.

    ```bash
    azd down --purge --force
    ```

---

Congratulations! You have developed an agent in Copilot Studio and integrated an MCP server.

👈 [06: Integrating MCP Server with Microsoft Agent Framework](./06-mcp-server-integration-with-maf.md) | [README](../README.md) 👉

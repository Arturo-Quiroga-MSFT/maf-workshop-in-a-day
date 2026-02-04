# 05: Developing MCP Server

In this session, we will develop a [Model Context Protocol (MCP) server](https://modelcontextprotocol.io) to integrate with the backend agent.

## Session Goals

- You will be able to develop an MCP server.
- You will be able to run the MCP server in a local HTTP environment.
- You will be able to deploy the MCP server to Azure cloud.
- You will be able to run the MCP server in a remote HTTP environment.
- You will be able to connect a local or remote MCP server to GitHub Copilot.

## Architecture

After completing this session, the following system will be created.

![Session Architecture](./images/step-05-architecture.png)

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
└── step-05/
    └── start/
        ├── .vscode/
        │   ├── mcp.http.local.json
        │   └── mcp.http.remote.json
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
> - `.vscode`: Configuration file directory for MCP server execution
> - `infra`: Bicep file directory for Azure cloud resource deployment
> - `MafWorkshop.McpTodo`: MCP server project for to-do list management

1. If you have a `workshop` directory from the previous exercise, delete it or rename it. Example: `workshop-step-04`
1. Open a terminal and run the following commands in order to create the exercise directory and copy the starting project.

    ```bash
    # zsh/bash
    rm -rf $REPOSITORY_ROOT/workshop && \
        mkdir -p $REPOSITORY_ROOT/workshop && \
        cp -a $REPOSITORY_ROOT/save-points/step-05/start/. $REPOSITORY_ROOT/workshop/
    ```

    ```powershell
    # PowerShell
    Remove-Item -Path $REPOSITORY_ROOT/workshop -Recurse -Force && `
        New-Item -Type Directory -Path $REPOSITORY_ROOT/workshop -Force && `
        Copy-Item -Path $REPOSITORY_ROOT/save-points/step-05/start/* -Destination $REPOSITORY_ROOT/workshop -Recurse -Force
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

## Configuring HTTP-based MCP Server

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Open the `./MafWorkshop.McpTodo/Program.cs` file and find the comment `// MCP 서버 추가하기` and add the following content. This registers the MCP service as an HTTP-type dependency object.

    ```csharp
    // MCP 서버 추가하기
    builder.Services.AddMcpServer()
                    .WithHttpTransport(o => o.Stateless = true)
                    .WithToolsFromAssembly(Assembly.GetEntryAssembly());
    ```

1. In the same file, find the comment `// MCP 엔드포인트 미들웨어 추가하기` and enter the following. This registers the endpoint for the MCP server.

    ```csharp
    // MCP 엔드포인트 미들웨어 추가하기
    app.MapMcp("/mcp");
    ```

## Adding Tools to the MCP Server

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Open the `./MafWorkshop.McpTodo/Program.cs` file and find the comment `// Todo Tool 추가하기` and add the following content. This creates tools that the LLM can utilize through this MCP server.

    ```csharp
    // Todo Tool 추가하기
    [McpServerToolType]
    public class TodoTool(ITodoRepository todo, ILogger<TodoTool> logger)
    {
        [McpServerTool(Name = "add_todo_item", Title = "Add a to-do item")]
        [Description("Adds a to-do item to database.")]
        public async Task<string> AddTodoItemAsync(
            [Description("The to-do item text")] string todoItemText
        )
        {
            var todoItem = new TodoItem { Text = todoItemText };
            await todo.AddTodoItemAsync(todoItem).ConfigureAwait(false);
    
            logger.LogInformation("Todo item added: '{todoItemText}' (ID: {Id})", todoItemText, todoItem.Id);
    
            return $"Todo item added: '{todoItemText}' (ID: {todoItem.Id})";
        }
    
        [McpServerTool(Name = "get_todo_items", Title = "Get a list of to-do items")]
        [Description("Gets a list of to-do items from database.")]
        public async Task<IEnumerable<string>> GetTodoItemsAsync()
        {
            var todoItems = await todo.GetAllTodoItemsAsync().ConfigureAwait(false);
    
            logger.LogInformation("Retrieved {Count} todo items.", todoItems.Count());
    
            return todoItems.Any()
                   ? todoItems.Select(p => $"ID: {p.Id}, Text: {p.Text}, Completed: {p.IsCompleted}")
                   : [ "No todo items found." ];
        }
    
        [McpServerTool(Name = "update_todo_item", Title = "Update a to-do item")]
        [Description("Updates a to-do item in the database.")]
        public async Task<string> UpdateTodoItemAsync(
            [Description("The to-do item ID")] int id,
            [Description("The to-do item text")] string todoItemText
        )
        {
            var todoItem = new TodoItem { Id = id, Text = todoItemText };
            var updated = await todo.UpdateTodoItemAsync(todoItem).ConfigureAwait(false);
            if (updated is null)
            {
                logger.LogWarning("Todo item with ID '{id}' not found.", id);
    
                return $"Todo item with ID '{id}' not found.";
            }
    
            logger.LogInformation("Updated todo item: '{id}' with text: '{todoItem}'", id, todoItem);
    
            return $"Todo item updated: '{id}' with text: '{todoItem}'";
        }
    
        [McpServerTool(Name = "complete_todo_item", Title = "Complete a to-do item")]
        [Description("Completes a to-do item in the database.")]
        public async Task<string> CompleteTodoItemAsync(
            [Description("The to-do item ID")] int id
        )
        {
            var todoItem = new TodoItem() { Id = id, IsCompleted = true };
            var completed = await todo.CompleteTodoItemAsync(todoItem).ConfigureAwait(false);
            if (completed is null)
            {
                logger.LogWarning("Todo item with ID '{id}' not found.", id);
    
                return $"Todo item with ID '{id}' not found.";
            }
    
            logger.LogInformation("Completed todo item: '{id}'", id);
    
            return $"Todo item completed: '{id}'";
        }
    
        [McpServerTool(Name = "delete_todo_item", Title = "Delete a to-do item")]
        [Description("Deletes a to-do item from the database.")]
        public async Task<string> DeleteTodoItemAsync(
            [Description("The to-do item ID")] int id
        )
        {
            var deleted = await todo.DeleteTodoItemAsync(id).ConfigureAwait(false);
            if (deleted is null)
            {
                logger.LogWarning("Todo item with ID '{id}' not found.", id);
    
                return $"Todo item with ID '{id}' not found.";
            }
    
            logger.LogInformation("Deleted todo item: '{id}'", id);
    
            return $"Todo item deleted: '{id}'";
        }
    }
    ```

## Connecting to GitHub Copilot from Local MCP Server

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Run the following command to create the `.vscode/mcp.json` file.

    ```bash
    # zsh/bash
    mkdir -p $REPOSITORY_ROOT/.vscode
    cp ./.vscode/mcp.http.local.json $REPOSITORY_ROOT/.vscode/mcp.json 
    ```

    ```powershell
    # PowerShell
    New-Item -Type Directory -Path $REPOSITORY_ROOT/.vscode -Force
    Copy-Item -Path ./.vscode/mcp.http.local.json -Destination $REPOSITORY_ROOT/.vscode/mcp.json -Force
    ```

1. Run the MCP server application.

    ```bash
    dotnet run --project ./MafWorkshop.McpTodo
    ```

1. Click on the right extension icon and look at the MCP Server section where you'll see the `todo-list` MCP server. Click on the gear icon and then click the `Start Server` menu to run the MCP server.

   ![GitHub Copilot - MCP Server Run](./images/step-05-image-01.png)

1. Open the GitHub Copilot window and verify that the `todo-list` MCP server is selected as shown below.

   ![GitHub Copilot - MCP Server Selection](./images/step-05-image-02.png)

1. In the GitHub Copilot window, send prompts similar to the following.

    ```text
    - Show me today's tasks
    - Add a 2pm meeting
    ```

   ![GitHub Copilot - MCP Server Run](./images/step-05-image-03.png)

1. Verify that GitHub Copilot successfully runs the `todo-list` MCP server and performs the desired tasks.

   ![GitHub Copilot - MCP Server Run Result](./images/step-05-image-04.png)

1. Click on the right extension icon and look at the MCP Server section where you'll see the `todo-list` MCP server. Click on the gear icon and then click the `Stop Server` menu to stop the MCP server.

   ![GitHub Copilot - MCP Server Stop](./images/step-05-image-05.png)

1. Press `CTRL`+`C` in the terminal to stop the application.

## Connecting to GitHub Copilot from Remote MCP Server

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

1. Run the following command to create the `.vscode/mcp.json` file.

    ```bash
    # zsh/bash
    mkdir -p $REPOSITORY_ROOT/.vscode
    cp ./.vscode/mcp.http.local.json $REPOSITORY_ROOT/.vscode/mcp.json 
    ```

    ```powershell
    # PowerShell
    New-Item -Type Directory -Path $REPOSITORY_ROOT/.vscode -Force
    Copy-Item -Path ./.vscode/mcp.http.local.json -Destination $REPOSITORY_ROOT/.vscode/mcp.json -Force
    ```

1. Click on the right extension icon and look at the MCP Server section where you'll see the `todo-list` MCP server. Click on the gear icon and then click the `Start Server` menu to run the MCP server.

   ![GitHub Copilot - MCP Server Run](./images/step-05-image-01.png)

   During the process of starting the server, you will be asked for the remote server address as shown below. Enter the remote MCP server address you obtained earlier.

   ![GitHub Copilot - MCP Server Run - Remote Server Address Input](./images/step-05-image-06.png)

1. Open the GitHub Copilot window and verify that the `todo-list` MCP server is selected as shown below.

   ![GitHub Copilot - MCP Server Selection](./images/step-05-image-02.png)

1. In the GitHub Copilot window, send prompts similar to the following.

    ```text
    - Show me today's tasks
    - Add a 2pm meeting
    ```

   ![GitHub Copilot - MCP Server Run](./images/step-05-image-03.png)

1. Verify that GitHub Copilot successfully runs the `todo-list` MCP server and performs the desired tasks.

   ![GitHub Copilot - MCP Server Run Result](./images/step-05-image-04.png)

1. Click on the right extension icon and look at the MCP Server section where you'll see the `todo-list` MCP server. Click on the gear icon and then click the `Stop Server` menu to stop the MCP server.

   ![GitHub Copilot - MCP Server Stop](./images/step-05-image-05.png)

1. Run the following command to delete all the applications you just deployed.

    ```bash
    azd down --purge --force

## Verifying the Complete Result

The completed version of this session can be found at `$REPOSITORY_ROOT/save-points/step-05/complete`.

1. If you have a `workshop` directory from the previous exercise, delete it or rename it. Example: `workshop-step-05`
1. Open a terminal and run the following commands in order to create the exercise directory and copy the starting project.

    ```bash
    # zsh/bash
    mkdir -p $REPOSITORY_ROOT/workshop && \
        cp -a $REPOSITORY_ROOT/save-points/step-04/complete/. $REPOSITORY_ROOT/workshop/
    ```

    ```powershell
    # PowerShell
    New-Item -Type Directory -Path $REPOSITORY_ROOT/workshop -Force && `
        Copy-Item -Path $REPOSITORY_ROOT/save-points/step-04/complete/* -Destination $REPOSITORY_ROOT/workshop -Recurse -Force
    ```

1. Navigate to the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Follow the [Connecting to GitHub Copilot from Local MCP Server](#connecting-to-github-copilot-from-local-mcp-server) section.
1. Follow the [Connecting to GitHub Copilot from Remote MCP Server](#connecting-to-github-copilot-from-remote-mcp-server) section.

---

Congratulations! You have developed an MCP server for use with agents. Now proceed to the next step!

👈 [04: Orchestrating Frontend Web UI and Backend Agent with Aspire](./04-aspire-orchestration.md) | [06: Integrating MCP Server with Microsoft Agent Framework](./06-mcp-server-integration-with-maf.md) 👉

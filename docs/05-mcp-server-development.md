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

## 시작 프로젝트 복사

이 워크샵을 위해 필요한 시작 프로젝트를 준비해 뒀습니다. 시작 프로젝트의 프로젝트 구조는 아래와 같습니다.

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

> 프로젝트 소개:
>
> - `.vscode`: MCP 서버 실행용 설정 파일 디렉토리
> - `infra`: Azure 클라우드 리소스 배포용 bicep 파일 디렉토리
> - `MafWorkshop.McpTodo`: To-do 리스트 관리용 MCP 서버 프로젝트

1. 앞서 실습한 `workshop` 디렉토리가 있다면 삭제하거나 다른 이름으로 바꿔주세요. 예) `workshop-step-04`
1. 터미널을 열고 아래 명령어를 차례로 실행시켜 실습 디렉토리를 만들고 시작 프로젝트를 복사합니다.

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

1. MCP 서버 애플리케이션을 실행합니다.

    ```bash
    dotnet run --project ./MafWorkshop.McpTodo
    ```

1. 오른쪽 익스텐션 아이콘을 클릭한 후 MCP 서버 섹션을 보면 `todo-list` MCP 서버가 보입니다. 톱니바퀴 모양을 클릭한 후 `Start Server` 메뉴를 클릭해서 MCP 서버를 실행시킵니다.

   ![GitHub Copilot - MCP 서버 실행](./images/step-05-image-01.png)

1. GitHub Copilot 창을 열어 아래와 같이 `todo-list` MCP 서버를 선택했는지 확인합니다.

   ![GitHub Copilot - MCP 서버 선택](./images/step-05-image-02.png)

1. GitHub Copilot 창에서 아래와 비슷한 프롬프트를 전송합니다.

    ```text
    - 오늘 할 일 보여줘
    - 오후 2시 미팅 추가해줘
    ```

   ![GitHub Copilot - MCP 서버 실행](./images/step-05-image-03.png)

1. GitHub Copilot이 `todo-list` MCP 서버를 잘 실행시켜 원하는 작업을 수행했는지 확인합니다.

   ![GitHub Copilot - MCP 서버 실행 결과](./images/step-05-image-04.png)

1. 오른쪽 익스텐션 아이콘을 클릭한 후 MCP 서버 섹션을 보면 `todo-list` MCP 서버가 보입니다. 톱니바퀴 모양을 클릭한 후 `Stop Server` 메뉴를 클릭해서 MCP 서버를 종료합니다.

   ![GitHub Copilot - MCP 서버 종료](./images/step-05-image-05.png)

1. 터미널에서 `CTRL`+`C` 키를 눌러 애플리케이션 실행을 종료합니다.

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

1. 오른쪽 익스텐션 아이콘을 클릭한 후 MCP 서버 섹션을 보면 `todo-list` MCP 서버가 보입니다. 톱니바퀴 모양을 클릭한 후 `Start Server` 메뉴를 클릭해서 MCP 서버를 실행시킵니다.

   ![GitHub Copilot - MCP 서버 실행](./images/step-05-image-01.png)

   서버를 실행시키는 과정에서 아래와 같이 리모트 서버의 주소를 물어봅니다. 이 때 앞서 확인했던 리모트 MCP 서버의 주소를 입력하세요.

   ![GitHub Copilot - MCP 서버 실행 - 리모트 서버 주소 입력](./images/step-05-image-06.png)

1. GitHub Copilot 창을 열어 아래와 같이 `todo-list` MCP 서버를 선택했는지 확인합니다.

   ![GitHub Copilot - MCP 서버 선택](./images/step-05-image-02.png)

1. GitHub Copilot 창에서 아래와 비슷한 프롬프트를 전송합니다.

    ```text
    - 오늘 할 일 보여줘
    - 오후 2시 미팅 추가해줘
    ```

   ![GitHub Copilot - MCP 서버 실행](./images/step-05-image-03.png)

1. GitHub Copilot이 `todo-list` MCP 서버를 잘 실행시켜 원하는 작업을 수행했는지 확인합니다.

   ![GitHub Copilot - MCP 서버 실행 결과](./images/step-05-image-04.png)

1. 오른쪽 익스텐션 아이콘을 클릭한 후 MCP 서버 섹션을 보면 `todo-list` MCP 서버가 보입니다. 톱니바퀴 모양을 클릭한 후 `Stop Server` 메뉴를 클릭해서 MCP 서버를 종료합니다.

   ![GitHub Copilot - MCP 서버 종료](./images/step-05-image-05.png)

1. 아래 명령어를 실행시켜 방금 배포한 애플리케이션을 모두 삭제합니다.

    ```bash
    azd down --purge --force

## 완성본 결과 확인

이 세션의 완성본은 `$REPOSITORY_ROOT/save-points/step-05/complete`에서 확인할 수 있습니다.

1. 앞서 실습한 `workshop` 디렉토리가 있다면 삭제하거나 다른 이름으로 바꿔주세요. 예) `workshop-step-05`
1. 터미널을 열고 아래 명령어를 차례로 실행시켜 실습 디렉토리를 만들고 시작 프로젝트를 복사합니다.

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

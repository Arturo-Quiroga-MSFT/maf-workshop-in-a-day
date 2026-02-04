# 06: Integrating MCP Server with Microsoft Agent Framework

In this session, we will integrate the [MCP server](./05-mcp-server-development.md) we created earlier with the backend agent in Microsoft Agent Framework.

## Session Goals

- You will be able to integrate an MCP server with Microsoft Agent Framework.
- You will be able to orchestrate the frontend web UI, backend agent, LLM connection, and MCP server using Aspire.
- You will be able to deploy the entire application to Azure cloud.

## Architecture

After completing this session, the following system will be created.

![Session Architecture](./images/step-06-architecture.png)

## Prerequisites

- It is assumed that you have completed all the development environment setup in the previous [00: Development Environment Setup](./00-setup.md).

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
└── step-06/
    └── start/
        ├── MafWorkshop.sln
        ├── MafWorkshop.Agent/
        │   ├── Properties/
        │   │   └── launchSettings.json
        │   ├── Program.cs
        │   ├── appsettings.json
        │   └── MafWorkshop.Agent.csproj
        ├── MafWorkshop.McpTodo/
        │   ├── Properties/
        │   │   └── launchSettings.json
        │   ├── TodoDbContext.cs
        │   ├── TodoTool.cs
        │   ├── Program.cs
        │   ├── appsettings.json
        │   └── MafWorkshop.McpTodo.csproj
        ├── MafWorkshop.WebUI/
        │   ├── Properties/
        │   │   └── launchSettings.json
        │   ├── Components/
        │   │   └── < Razor component files >
        │   ├── wwwroot/
        │   │   └── < HTML/CSS/JS files >
        │   ├── Program.cs
        │   ├── appsettings.json
        │   └── MafWorkshop.WebUI.csproj
        ├── MafWorkshop.AppHost/
        │   ├── Properties/
        │   │   └── launchSettings.json
        │   ├── LlmResourceFactory.cs
        │   ├── Program.cs
        │   ├── appsettings.json
        │   └── MafWorkshop.AppHost.csproj
        └── MafWorkshop.ServiceDefaults/
            ├── Extension.cs
            └── MafWorkshop.ServiceDefaults.csproj
```

> 프로젝트 소개:
>
> - `MafWorkshop.Agent`: 백엔드 에이전트 애플리케이션 프로젝트
> - `MafWorkshop.McpTodo`: To-do 리스트 관리용 MCP 서버 프로젝트
> - `MafWorkshop.WebUI`: 프론트엔드 웹 UI 애플리케이션 프로젝트
> - `MafWorkshop.AppHost`: Aspire 오케스트레이션 프로젝트
> - `MafWorkshop.ServiceDefaults`: Aspire Observability 및 Traceability 확장 프로젝트

1. 앞서 실습한 `workshop` 디렉토리가 있다면 삭제하거나 다른 이름으로 바꿔주세요. 예) `workshop-step-05`
1. 터미널을 열고 아래 명령어를 차례로 실행시켜 실습 디렉토리를 만들고 시작 프로젝트를 복사합니다.

    ```bash
    # zsh/bash
    rm -rf $REPOSITORY_ROOT/workshop && \
        mkdir -p $REPOSITORY_ROOT/workshop && \
        cp -a $REPOSITORY_ROOT/save-points/step-06/start/. $REPOSITORY_ROOT/workshop/
    ```

    ```powershell
    # PowerShell
    Remove-Item -Path $REPOSITORY_ROOT/workshop -Recurse -Force && `
        New-Item -Type Directory -Path $REPOSITORY_ROOT/workshop -Force && `
        Copy-Item -Path $REPOSITORY_ROOT/save-points/step-06/start/* -Destination $REPOSITORY_ROOT/workshop -Recurse -Force
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

1. Run the Aspire orchestration application.

    ```bash
    dotnet watch run --project ./MafWorkshop.AppHost
    ```

1. Verify that the web browser opens automatically and shows the Aspire dashboard page like below.

   ![Aspire Dashboard Page - Before MCP Server Integration](./images/step-06-image-01.png)

1. Press `CTRL`+`C` in the terminal to stop the application.

## Integrating Observability and Traceability Tools - MCP Server

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Run the following command to add Observability and Traceability tools.

    ```bash
    dotnet add ./MafWorkshop.McpTodo reference ./MafWorkshop.ServiceDefaults
    ```

1. Open the `./MafWorkshop.McpTodo/Program.cs` file and find the comment `// Observability 및 Traceability를 위한 Service Defaults 추가하기` and add the following content. This registers various Observability and Traceability related instances as dependency objects.

    ```csharp
    // Observability 및 Traceability를 위한 Service Defaults 추가하기
    builder.AddServiceDefaults();
    ```

1. 같은 파일에서 `// Observability 및 Traceability를 위한 미들웨어 설정하기` 주석을 찾아 아래와 같이 입력합니다. 서비스의 가용성 확인을 위한 헬스체크 엔드포인트를 추가하는 미들웨어입니다.

    ```csharp
    // Observability 및 Traceability를 위한 미들웨어 설정하기
    app.MapDefaultEndpoints();
    ```

## Aspire 오케스트레이션 구성 - 호스트

1. 워크샵 디렉토리에 있는지 다시 한 번 확인합니다.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. 아래 명령어를 실행시켜 오케스트레이션을 위한 백엔드 및 프론트엔드 애플리케이션을 추가합니다.

    ```bash
    dotnet add ./MafWorkshop.AppHost reference ./MafWorkshop.McpTodo
    ```

1. `./MafWorkshop.AppHost/AppHost.cs` 파일을 열고 `// MCP Todo 서버 프로젝트 추가하기` 주석을 찾아 아래 내용을 추가합니다. MCP 서버 앱을 `mcptodo`라는 리소스로 선언합니다.

    ```csharp
    // MCP Todo 서버 프로젝트 추가하기
    var mcptodo = builder.AddProject<Projects.MafWorkshop_McpTodo>("mcptodo")
                         .WithExternalHttpEndpoints();
    ```

1. 같은 파일에서 `// 백엔드 에이전트 프로젝트 수정하기` 주석을 찾아 아래와 같이 변경합니다. 백엔드 에이전트 리소스인 `agent`에 방금 작성한 `mcptodo` 리소스를 연결합니다.

   **변경전:**

    ```csharp
    // 백엔드 에이전트 프로젝트 수정하기
    var agent = builder.AddProject<Projects.MafWorkshop_Agent>("agent")
                       .WithExternalHttpEndpoints()
                       .WithLlmReference(builder.Configuration);
    ```

   **After change:**

    ```csharp
    // 백엔드 에이전트 프로젝트 수정하기
    var agent = builder.AddProject<Projects.MafWorkshop_Agent>("agent")
                       .WithExternalHttpEndpoints()
                       .WithLlmReference(builder.Configuration)
                       .WithReference(mcptodo)
                       .WaitFor(mcptodo);
    ```

## Aspire Orchestration Configuration - Backend Agent

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Open the `./MafWorkshop.Agent/Program.cs` file and find the comment `// HttpClientFactory 등록하기` and add the following content. Since the MCP server connection is handled by the Aspire `AppHost` project, register the `IHttpClientFactory` instance received from Aspire as a dependency object.

    ```csharp
    // HttpClientFactory 등록하기
    builder.Services.AddHttpClient("mcptodo", client =>
    {
        client.BaseAddress = new Uri("https+http://mcptodo");
    });
    ```

1. In the same file, find the comment `// MCP 클라이언트 등록하기` and enter the following. This registers the `McpClient` instance along with the `IHttpClientFactory` instance added earlier as dependency objects.

    ```csharp
    // MCP 클라이언트 등록하기
    builder.Services.AddSingleton<McpClient>(sp =>
    {
        var loggerFactory = sp.GetRequiredService<ILoggerFactory>();
        var httpClient = sp.GetRequiredService<IHttpClientFactory>()
                           .CreateClient("mcptodo");
    
        var clientTransportOptions = new HttpClientTransportOptions()
        {
            Endpoint = new Uri($"{httpClient.BaseAddress!.ToString().Replace("+http", string.Empty).TrimEnd('/')}/mcp")
        };
        var clientTransport = new HttpClientTransport(clientTransportOptions, httpClient, loggerFactory);
    
        var clientOptions = new McpClientOptions()
        {
            ClientInfo = new Implementation()
            {
                Name = "MCP Todo Client",
                Version = "1.0.0",
            }
        };
    
        return McpClient.CreateAsync(clientTransport, clientOptions, loggerFactory).GetAwaiter().GetResult();
    });
    ```

1. In the same file, find the comment `// Manager 에이전트 추가하기` and enter the following. Here we add a **Manager** agent that manages the to-do list, and this agent is configured to use the tools provided by the MCP server as agent tools.

    ```csharp
    // Manager 에이전트 추가하기
    builder.AddAIAgent(
        name: "manager",
        createAgentDelegate: (sp, key) =>
        {
            var chatClient = sp.GetRequiredService<IChatClient>();
            var mcpClient = sp.GetRequiredService<McpClient>();
            var tools = mcpClient.ListToolsAsync().GetAwaiter().GetResult();
            var agent = new ChatClientAgent(
                chatClient: chatClient,
                name: key,
                instructions: """
                    You manage my todo list items.
                    When I ask for the list, provide all the items in a numbered format with their complete status.
                    When I give you a new todo item, add it to the list.
                    When I give you an updated todo item, update it in the list.
                    When I ask you to mark an item as done, mark it as completed.
                    When I ask you to remove an item, delete it from the list.
                    When I ask you to clear the list, remove all items.
                    """,
                tools: [.. tools]
            );
    
            return agent;
        }
    );
    ```

1. In the same file, find the comment `// AG-UI 미들웨어 설정하기` and enter the following. This configures communication with the frontend web UI through the `/ag-ui` endpoint.

    ```csharp
    // AG-UI 미들웨어 설정하기
    app.MapAGUI(
        pattern: "ag-ui",
        aiAgent: app.Services.GetRequiredKeyedService<AIAgent>("manager")
    );
    ```

## Building and Running the Application

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Build the entire project.

    ```bash
    dotnet restore && dotnet build
    ```

1. Run the Aspire orchestration application.

    ```bash
    dotnet watch run --project ./MafWorkshop.AppHost
    ```

1. Verify that the web browser opens automatically and shows the Aspire dashboard page like below.

   ![Aspire Dashboard Page - After MCP Server Integration](./images/step-06-image-02.png)

1. Click on the backend agent app link and verify that the Dev UI screen displays correctly. Then, select the Manager agent to verify it works properly.

   ![Dev UI Page - Manager Agent](./images/step-06-image-03.png)

1. Click on the frontend web UI app link and verify that the chat UI screen displays correctly. Then, enter a message to verify the output is correct.

   ![Web UI Page](./images/step-06-image-04.png)

1. Press `CTRL`+`C` in the terminal to stop the application.

## Deploying and Running the Application

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
   - `? Enter a value for the 'apiKey' infrastructure secured parameter:` 👉 Enter API key value
   - `? Enter a value for the 'location' infrastructure parameter:` 👉 Select region (e.g., `Korea Central`)

   Wait a moment and you can verify that the Azure Container Apps instances for the frontend web UI, backend agent, and MCP server have been created.

   ![Application Deployment Result](./images/step-06-image-05.png)

1. Click on the `webui` link in the screenshot above and when the web UI screen appears, enter prompts similar to the following and verify the results.

    ```text
    - Show me today's tasks
    - Add a 2pm meeting
    ```

   ![Application Execution Result](./images/step-06-image-06.png)

1. Run the following command to delete all the applications you just deployed.

    ```bash
    azd down --purge --force

## Verifying the Complete Result

The completed version of this session can be found at `$REPOSITORY_ROOT/save-points/step-06/complete`.

1. If you have a `workshop` directory from the previous exercise, delete it or rename it. Example: `workshop-step-06`
1. Open a terminal and run the following commands in order to create the exercise directory and copy the starting project.

    ```bash
    # zsh/bash
    mkdir -p $REPOSITORY_ROOT/workshop && \
        cp -a $REPOSITORY_ROOT/save-points/step-06/complete/. $REPOSITORY_ROOT/workshop/
    ```

    ```powershell
    # PowerShell
    New-Item -Type Directory -Path $REPOSITORY_ROOT/workshop -Force && `
        Copy-Item -Path $REPOSITORY_ROOT/save-points/step-06/complete/* -Destination $REPOSITORY_ROOT/workshop -Recurse -Force
    ```

1. Navigate to the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Follow the [Building and Running the Application](#building-and-running-the-application) section.
1. Follow the [Deploying and Running the Application](#deploying-and-running-the-application) section.

---

Congratulations! You have integrated an MCP server with the backend agent. Now proceed to the next step!

👈 [05: Developing MCP Server](./05-mcp-server-development.md) | [07: Developing an Agent in Copilot Studio and Integrating MCP Server **(Optional)**](./07-mcp-server-integration-with-copilot-studio.md) 👉

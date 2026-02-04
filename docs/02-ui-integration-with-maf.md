# 02: Integrating Frontend UI with Microsoft Agent Framework

This session covers integrating a frontend web UI with the backend agent built with Microsoft Agent Framework using the [AG-UI protocol](https://docs.ag-ui.com/introduction).

## Session Goals

- Connect a frontend UI to Microsoft Agent Framework using the AG-UI protocol.

## Architecture

Upon completing this session, you will have built the following system.

![Session Architecture](./images/step-02-architecture.png)

## Prerequisites

This assumes you have completed all the development environment setup from [00: Development Environment Setup](./00-setup.md).

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

## Copying the Starting Project

We have prepared the necessary starting project for this workshop. The project structure of the starting project is as follows:

```text
save-points/
└── step-02/
    └── start/
        ├── MafWorkshop.sln
        ├── MafWorkshop.Agent/
        │   ├── Properties/
        │   │   └── launchSettings.json
        │   ├── Program.cs
        │   ├── appsettings.json
        │   └── MafWorkshop.Agent.csproj
        └── MafWorkshop.WebUI/
            ├── Properties/
            │   └── launchSettings.json
            ├── Components/
            │   └── < Razor component files >
            ├── wwwroot/
            │   └── < HTML/CSS/JS files >
            ├── Program.cs
            ├── appsettings.json
            └── MafWorkshop.WebUI.csproj
```

> Project overview:
>
> - `MafWorkshop.Agent`: Backend agent application project
> - `MafWorkshop.WebUI`: Frontend web UI application project

1. If you have the `workshop` directory from the previous exercise, delete it or rename it. For example: `workshop-step-01`
1. Open a terminal and run the following commands in order to create the workshop directory and copy the starting project.

    ```bash
    # zsh/bash
    rm -rf $REPOSITORY_ROOT/workshop && \
        mkdir -p $REPOSITORY_ROOT/workshop && \
        cp -a $REPOSITORY_ROOT/save-points/step-02/start/. $REPOSITORY_ROOT/workshop/
    ```

    ```powershell
    # PowerShell
    Remove-Item -Path $REPOSITORY_ROOT/workshop -Recurse -Force && `
        New-Item -Type Directory -Path $REPOSITORY_ROOT/workshop -Force && `
        Copy-Item -Path $REPOSITORY_ROOT/save-points/step-02/start/* -Destination $REPOSITORY_ROOT/workshop -Recurse -Force
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

1. Open another terminal and run the frontend UI application.

    ```bash
    dotnet watch run --project ./MafWorkshop.WebUI
    ```

1. Verify that the web browser opens automatically and shows a chat UI page as below.

   ![Web UI Page](./images/step-02-image-01.png)

   Enter any message and verify that a fake response appears as shown below.

   ![Web UI Page - Fake Response](./images/step-02-image-02.png)

1. Press `CTRL`+`C` in the terminal to stop the application execution.

## Integrating AG-UI Protocol with Backend Agent App

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Open the `./MafWorkshop.Agent/appsettings.json` file and verify that the `LlmProvider` value is `GitHubModels`. If it's set to a different value, change it to `GitHubModels`.

    ```jsonc
    {
      "LlmProvider": "GitHubModels"
    }
    ```

   > **If you have an Azure subscription**, try changing `GitHubModels` to `AzureOpenAI`.

1. Open the `./MafWorkshop.Agent/Program.cs` file and find the comment `// AG-UI 등록하기` and add the following content. Directly register service instances that enable AG-UI service usage in the agent app as dependency objects without implementing separate logic.

    ```csharp
    // AG-UI 등록하기
    builder.Services.AddAGUI();
    ```

1. In the same file, find the comment `// AG-UI 미들웨어 설정하기` and enter the following. Through this middleware, add the `/ag-ui` endpoint to the backend agent app and connect this endpoint to the Writer agent.

    ```csharp
    // AG-UI 미들웨어 설정하기
    app.MapAGUI(
        pattern: "ag-ui",
        aiAgent: app.Services.GetRequiredKeyedService<AIAgent>("writer")
    );
    ```

## Integrating AG-UI Protocol with Frontend UI App

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Open the `./MafWorkshop.WebUI/appsettings.json` file and verify that the `AgentEndpoints` section has the following values. If not, adjust them as shown below.

    ```jsonc
    {
      "AgentEndpoints": {
        "Https": "https://localhost:45097",
        "http": "http://localhost:5097"
      }
    }
    ```

1. Open the `./MafWorkshop.WebUI/Program.cs` file and find the comment `// HttpClientFactory 등록하기` and add the following content. Register an `HttpClient` instance with the name `agent` to find the backend agent application.

    ```csharp
    // HttpClientFactory 등록하기
    builder.Services.AddHttpClient("agent", client =>
    {
        var endpoint = builder.Environment.IsDevelopment() == true
            ? builder.Configuration["AgentEndpoints:Http"]
            : builder.Configuration["AgentEndpoints:Https"];
        client.BaseAddress = new Uri(endpoint!);
    });
    ```

1. In the same file, find the comment `// AG-UI 연동 IChatClient 인스턴스 등록하기` and verify it looks like the following. Currently it's connected to `FakeChatClient` which generates fake responses.

    ```csharp
    // AG-UI 연동 IChatClient 인스턴스 등록하기
    builder.Services.AddChatClient(new FakeChatClient());
    ```

   Modify it as follows. Connect the `HttpClient` instance with the name `agent` registered earlier to the `/ag-ui` endpoint of the backend agent app.

    ```csharp
    // AG-UI 연동 IChatClient 인스턴스 등록하기
    builder.Services.AddChatClient(sp => new AGUIChatClient(
        httpClient: sp.GetRequiredService<IHttpClientFactory>().CreateClient("agent"),
        endpoint: "ag-ui")
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

1. Run the backend agent application.

    ```bash
    dotnet run --project ./MafWorkshop.Agent
    ```

1. Open another terminal and run the frontend UI application.

    ```bash
    dotnet watch run --project ./MafWorkshop.WebUI
    ```

1. Verify that the web browser opens automatically and shows a chat UI page as below.

   ![Web UI Page](./images/step-02-image-01.png)

   Enter any sentence and check the results.

   ![Web UI Page - Results Verification](./images/step-02-image-03.png)

1. Press `CTRL`+`C` in each terminal to stop all application execution.

## Verifying the Complete Result

The completed version of this session can be found at `$REPOSITORY_ROOT/save-points/step-02/complete`.

1. If you have the `workshop` directory from the previous exercise, delete it or rename it. For example: `workshop-step-02`
1. Open a terminal and run the following commands in order to create the workshop directory and copy the starting project.

    ```bash
    # zsh/bash
    mkdir -p $REPOSITORY_ROOT/workshop && \
        cp -a $REPOSITORY_ROOT/save-points/step-02/complete/. $REPOSITORY_ROOT/workshop/
    ```

    ```powershell
    # PowerShell
    New-Item -Type Directory -Path $REPOSITORY_ROOT/workshop -Force && `
        Copy-Item -Path $REPOSITORY_ROOT/save-points/step-02/complete/* -Destination $REPOSITORY_ROOT/workshop -Recurse -Force
    ```

1. Move to the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Follow the [Building and Running the Application](#building-and-running-the-application) section.

---

Congratulations! You have connected the frontend to the agent backend using the AG-UI protocol. Now proceed to the next step!

👈 [01: Building a Single Agent with Microsoft Agent Framework](./01-single-agent-with-maf.md) | [03: Building Multi-Agent with Microsoft Agent Framework](./03-multi-agent-with-maf.md) 👉

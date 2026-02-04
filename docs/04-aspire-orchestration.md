# 04: Orchestrating Frontend Web UI and Backend Agent with Aspire

This session covers orchestrating the frontend web UI app and backend agent app developed earlier using [Aspire](https://aspire.dev).

## Session Goals

- Orchestrate frontend web UI, backend agent, and LLM connections using Aspire.
- Configure cloud-native features such as Observability and Traceability through Aspire.
- Deploy the entire application to Azure cloud.

## Architecture

Upon completing this session, you will have built the following system.

![Session Architecture](./images/step-04-architecture.png)

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
└── step-04/
    └── start/
        ├── MafWorkshop.sln
        ├── MafWorkshop.Agent/
        │   ├── Properties/
        │   │   └── launchSettings.json
        │   ├── Program.cs
        │   ├── appsettings.json
        │   └── MafWorkshop.Agent.csproj
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
        │   ├── Program.cs
        │   ├── appsettings.json
        │   └── MafWorkshop.AppHost.csproj
        └── MafWorkshop.ServiceDefaults/
            ├── Extension.cs
            └── MafWorkshop.ServiceDefaults.csproj
```

> Project overview:
>
> - `MafWorkshop.Agent`: Backend agent application project
> - `MafWorkshop.WebUI`: Frontend web UI application project
> - `MafWorkshop.AppHost`: Aspire orchestration project
> - `MafWorkshop.ServiceDefaults`: Aspire Observability and Traceability extension project

1. If you have the `workshop` directory from the previous exercise, delete it or rename it. For example: `workshop-step-03`
1. Open a terminal and run the following commands in order to create the workshop directory and copy the starting project.

    ```bash
    # zsh/bash
    rm -rf $REPOSITORY_ROOT/workshop && \
        mkdir -p $REPOSITORY_ROOT/workshop && \
        cp -a $REPOSITORY_ROOT/save-points/step-04/start/. $REPOSITORY_ROOT/workshop/
    ```

    ```powershell
    # PowerShell
    Remove-Item -Path $REPOSITORY_ROOT/workshop -Recurse -Force && `
        New-Item -Type Directory -Path $REPOSITORY_ROOT/workshop -Force && `
        Copy-Item -Path $REPOSITORY_ROOT/save-points/step-04/start/* -Destination $REPOSITORY_ROOT/workshop -Recurse -Force
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

1. Run the backend agent application.

    ```bash
    dotnet run --project ./MafWorkshop.Agent
    ```

1. Open another terminal and run the frontend UI application.

    ```bash
    dotnet watch run --project ./MafWorkshop.WebUI
    ```

1. Verify that the web browser opens automatically and shows a chat UI page as below.

   ![Web UI Page](./images/step-04-image-01.png)

   Enter any sentence and check the results.

   ![Web UI Page - Results Verification](./images/step-04-image-02.png)

1. Press `CTRL`+`C` in each terminal to stop all application execution.

## Integrating Observability and Traceability Tools - Backend Agent

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Run the following command to add Observability and Traceability tools.

    ```bash
    dotnet add ./MafWorkshop.Agent reference ./MafWorkshop.ServiceDefaults
    ```

1. Open the `./MafWorkshop.Agent/Program.cs` file and find the comment `// Observability 및 Traceability를 위한 Service Defaults 추가하기` and add the following content. This registers various Observability and Traceability related instances as dependency objects.

    ```csharp
    // Observability 및 Traceability를 위한 Service Defaults 추가하기
    builder.AddServiceDefaults();
    ```

1. In the same file, find the comment `// Observability 및 Traceability를 위한 미들웨어 설정하기` and enter the following. This middleware adds a health check endpoint to verify service availability.

    ```csharp
    // Observability 및 Traceability를 위한 미들웨어 설정하기
    app.MapDefaultEndpoints();
    ```

## Integrating Observability and Traceability Tools - Frontend Web UI

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Run the following command to add Observability and Traceability tools.

    ```bash
    dotnet add ./MafWorkshop.WebUI reference ./MafWorkshop.ServiceDefaults
    ```

1. Open the `./MafWorkshop.WebUI/Program.cs` file and find the comment `// Observability 및 Traceability를 위한 Service Defaults 추가하기` and add the following content. Similar to the backend agent app, this registers various Observability and Traceability related instances as dependency objects.

    ```csharp
    // Observability 및 Traceability를 위한 Service Defaults 추가하기
    builder.AddServiceDefaults();
    ```

1. In the same file, find the comment `// Observability 및 Traceability를 위한 미들웨어 설정하기` and enter the following. Similar to the backend agent app, this middleware adds a health check endpoint to verify service availability.

    ```csharp
    // Observability 및 Traceability를 위한 미들웨어 설정하기
    app.MapDefaultEndpoints();
    ```

## Aspire Orchestration Configuration - Host

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Run the following commands to add the backend and frontend applications for orchestration.

    ```bash
    dotnet add ./MafWorkshop.AppHost reference ./MafWorkshop.Agent
    dotnet add ./MafWorkshop.AppHost reference ./MafWorkshop.WebUI
    ```

1. Run the following command to install the host packages for Aspire orchestration.

    ```bash
    dotnet add ./MafWorkshop.AppHost package Aspire.Hosting.GitHub.Models
    dotnet add ./MafWorkshop.AppHost package Aspire.Hosting.OpenAI
    ```

1. Open the `./MafWorkshop.AppHost/AppHost.cs` file and find the comment `// LlmResourceFactory 클래스 추가하기` and add the following content. You can think of it as similar to the `ChatClientFactory` class we implemented in the backend agent app. However, the difference is that the `ChatClientFactory` class directly creates `IChatClient` instances, while the `LlmResourceFactory` class creates resource references.

    ```csharp
    // LlmResourceFactory 클래스 추가하기
    public static class LlmResourceFactory
    {
        public static IResourceBuilder<ProjectResource> WithLlmReference(this IResourceBuilder<ProjectResource> source, IConfiguration config)
        {
            var provider = config["LlmProvider"] ?? throw new InvalidOperationException("Missing configuration: LlmProvider");
            source = provider switch
            {
                "GitHubModels" => AddGitHubModelsResource(source, config),
                "AzureOpenAI" => AddAzureOpenAIResource(source, config),
                _ => throw new NotSupportedException($"The specified LLM provider '{provider}' is not supported.")
            };
    
            return source;
        }
    
        private static IResourceBuilder<ProjectResource> AddGitHubModelsResource(IResourceBuilder<ProjectResource> source, IConfiguration config)
        {
            var provider = config["LlmProvider"];
    
            var github = config.GetSection("GitHub");
            var endpoint = github["Endpoint"] ?? throw new InvalidOperationException("Missing configuration: GitHub:Endpoint");
            var token = github["Token"] ?? throw new InvalidOperationException("Missing configuration: GitHub:Token");
            var model = github["Model"] ?? throw new InvalidOperationException("Missing configuration: GitHub:Model");
    
            Console.WriteLine();
            Console.WriteLine($"\tUsing {provider}: {model}");
            Console.WriteLine();
    
            var apiKey = source.ApplicationBuilder
                               .AddParameter(name: "apiKey", value: token, secret: true);
            var chat = source.ApplicationBuilder
                             .AddGitHubModel(name: "chat", model: model)
                             .WithApiKey(apiKey);
    
            return source.WithReference(chat)
                         .WaitFor(chat);
        }
    
        private static IResourceBuilder<ProjectResource> AddAzureOpenAIResource(IResourceBuilder<ProjectResource> source, IConfiguration config)
        {
            var provider = config["LlmProvider"];
    
            var azure = config.GetSection("Azure:OpenAI");
            var endpoint = azure["Endpoint"] ?? throw new InvalidOperationException("Missing configuration: Azure:OpenAI:Endpoint");
            var accessKey = azure["ApiKey"] ?? throw new InvalidOperationException("Missing configuration: Azure:OpenAI:ApiKey");
            var deploymentName = azure["DeploymentName"] ?? throw new InvalidOperationException("Missing configuration: Azure:OpenAI:DeploymentName");
    
            Console.WriteLine();
            Console.WriteLine($"\tUsing {provider}: {deploymentName}");
            Console.WriteLine();
    
            var apiKey = source.ApplicationBuilder
                               .AddParameter(name: "apiKey", value: accessKey, secret: true);
            var chat = source.ApplicationBuilder
                             .AddOpenAI("openai")
                             .WithEndpoint($"{endpoint.TrimEnd('/')}/openai/v1/")
                             .WithApiKey(apiKey)
                             .AddModel(name: "chat", model: deploymentName);
    
            return source.WithReference(chat)
                         .WaitFor(chat);
        }
    }
    ```

1. In the same file, find the comment `// 백엔드 에이전트 프로젝트 추가하기` and enter the following. Declare the backend agent app as a resource called `agent` and connect the LLM resource through the `WithLlmReference` method we wrote earlier.

    ```csharp
    // 백엔드 에이전트 프로젝트 추가하기
    var agent = builder.AddProject<Projects.MafWorkshop_Agent>("agent")
                       .WithExternalHttpEndpoints()
                       .WithLlmReference(builder.Configuration);
    ```

1. In the same file, find the comment `// 프론트엔드 웹 UI 프로젝트 추가하기` and enter the following. Declare the frontend web UI app as a resource called `webUI` and connect the `agent` resource we just created.

    ```csharp
    // 프론트엔드 웹 UI 프로젝트 추가하기
    var webUI = builder.AddProject<Projects.MafWorkshop_WebUI>("webui")
                        .WithExternalHttpEndpoints()
                        .WithReference(agent)
                        .WaitFor(agent);
    ```

## Aspire Orchestration Configuration - Backend Agent

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Run the following command to install the client packages for Aspire orchestration in the backend agent application.

    ```bash
    dotnet add ./MafWorkshop.Agent package Aspire.OpenAI --prerelease
    ```

1. Open the `./MafWorkshop.Agent/appsettings.json` file and delete the `LlmProvider`, `Azure`, and `GitHub` sections. These are no longer needed as the Aspire `AppHost` project will handle this instead.
1. Open the `./MafWorkshop.Agent/Program.cs` file and find the comment `// IChatClient 인스턴스 생성하기` and delete the code directly below it. Similarly, this logic is no longer needed as the Aspire `AppHost` project will handle it instead.

   **Before deletion:**

    ```csharp
    // IChatClient 인스턴스 생성하기
    IChatClient? chatClient = ChatClientFactory.CreateChatClient(builder.Configuration);
    ```

    **After deletion:** Only the comment remains.

    ```csharp
    // IChatClient 인스턴스 생성하기
    ```

1. In the same file, find the comment `// IChatClient 인스턴스 등록하기` and change it as follows. This directly registers the `IChatClient` instance coming from Aspire as a dependency object.

   **Before change:**

    ```csharp
    // IChatClient 인스턴스 등록하기
    builder.Services.AddChatClient(chatClient);
    ```

   **After change:**

    ```csharp
    // IChatClient 인스턴스 등록하기
    builder.AddOpenAIClient("chat")
           .AddChatClient();
    ```

## Aspire Orchestration Configuration - Frontend Web UI

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Open the `./MafWorkshop.WebUI/appsettings.json` file and delete the `AgentEndpoints` section. This logic is no longer needed as the Aspire `AppHost` project will handle it instead.
1. Open the `./MafWorkshop.WebUI/Program.cs` file and find the comment `// HttpClientFactory 등록하기` and change it as follows. Similarly, since the Aspire `AppHost` project will handle this instead, register the `IHttpClientFactory` instance received from Aspire as a dependency object.

   **Before change:**

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

   **After change:**

    ```csharp
    // HttpClientFactory 등록하기
    builder.Services.AddHttpClient("agent", client =>
    {
        client.BaseAddress = new Uri("https+http://agent");
    });
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

   ![Aspire Dashboard Page - GitHub Models](./images/step-04-image-03.png)

1. Press `CTRL`+`C` in the terminal to stop the application.
1. Open the `./MafWorkshop.AppHost/appsettings.json` file and change the `LlmProvider` value to `AzureOpenAI`.

    ```jsonc
    {
      "LlmProvider": "AzureOpenAI"
    }
    ```

1. Run the Aspire orchestration app again and verify that the Aspire dashboard page like below appears.

   ![Aspire Dashboard Page - Azure OpenAI](./images/step-04-image-04.png)

1. Click on the backend agent app link and verify that the Dev UI screen displays correctly. Then, select the Publish workflow to verify it works properly.
1. Click on the frontend web UI app link and verify that the chat UI screen displays correctly. Then, enter a message to verify the output is correct.
1. Go to the Traces tab in the Aspire dashboard. Then select the `agent` resource and observe the entire data flow.

   ![Aspire Dashboard Page - Data Tracing #1](./images/step-04-image-05.png)

   Observe the data flow from the frontend web UI screen through the backend agent to the LLM service.

   ![Aspire Dashboard Page - Data Tracing #2](./images/step-04-image-06.png)

1. Press `CTRL`+`C` in the terminal to stop the application.

## Deploying and Running the Application

> **NOTE**: Proceed if you have been provided with an Azure subscription. Depending on the workshop, an Azure subscription may not be provided.

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Open the `./MafWorkshop.AppHost/appsettings.json` file and verify whether the `LlmProvider` value is `GitHubModels` or `AzureOpenAI`.
1. Run the following command to deploy the entire application.

    ```bash
    azd up
    ```

   When the following questions appear, enter appropriate values.

   - `? Enter a unique environment name:` 👉 Environment name (e.g., `mafworkshop-2026`)
   - `? Enter a value for the 'apiKey' infrastructure secured parameter:` 👉 Enter API key value - If `LlmProvider` is `GitHubModels`, enter the GitHub PAT value. If it's `AzureOpenAI`, enter the Azure OpenAI instance API key value.

   Wait a moment and you can verify that the Azure Container Apps instances for the frontend web UI and backend agent have been created.

   ![Application Deployment Result](./images/step-04-image-07.png)

1. Click on the `webui` link in the screenshot above and when the web UI screen appears, enter a message to verify the output is correct.

   ![Application Execution Result](./images/step-04-image-08.png)

1. Run the following command to delete all the applications you just deployed.

    ```bash
    azd down --purge --force
    ```

## Verifying the Complete Result

The completed version of this session can be found at `$REPOSITORY_ROOT/save-points/step-03/complete`.

1. If you have a `workshop` directory from the previous exercise, delete it or rename it. Example: `workshop-step-04`
1. Open a terminal and run the following commands in order to create the exercise directory and copy the starting project.

    ```bash
    # zsh/bash
    mkdir -p $REPOSITORY_ROOT/workshop && \
        cp -a $REPOSITORY_ROOT/save-points/step-03/complete/. $REPOSITORY_ROOT/workshop/
    ```

    ```powershell
    # PowerShell
    New-Item -Type Directory -Path $REPOSITORY_ROOT/workshop -Force && `
        Copy-Item -Path $REPOSITORY_ROOT/save-points/step-03/complete/* -Destination $REPOSITORY_ROOT/workshop -Recurse -Force
    ```

1. Navigate to the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Follow the [Building and Running the Application](#building-and-running-the-application) section.
1. Follow the [Deploying and Running the Application](#deploying-and-running-the-application) section.

---

Congratulations! You have orchestrated the frontend web UI, backend agent app, and various LLMs all with Aspire. Now proceed to the next step!

👈 [03: Developing Multi-Agent with Microsoft Agent Framework](./03-multi-agent-with-maf.md) | [05: Developing MCP Server](./05-mcp-server-development.md) 👉

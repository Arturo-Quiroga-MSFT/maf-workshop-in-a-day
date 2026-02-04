# 01: Building a Single Agent with Microsoft Agent Framework

This session covers developing a single agent backend application using Microsoft Agent Framework.

## Session Goals

- Connect various LLMs to Microsoft Agent Framework.
- Attach a single agent to Microsoft Agent Framework.
- Visualize the flow of agents running in Microsoft Agent Framework.

## Architecture

Upon completing this session, you will have built the following system.

![Session Architecture](./images/step-01-architecture.png)

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
└── step-01/
    └── start/
        ├── MafWorkshop.sln
        └── MafWorkshop.Agent/
            ├── Properties/
            │   └── launchSettings.json
            ├── Program.cs
            ├── appsettings.json
            └── MafWorkshop.Agent.csproj
```

> Project overview:
>
> - `MafWorkshop.Agent`: Backend agent application project

1. Open a terminal and run the following commands in order to create the workshop directory and copy the starting project.

    ```bash
    # zsh/bash
    mkdir -p $REPOSITORY_ROOT/workshop && \
        cp -a $REPOSITORY_ROOT/save-points/step-01/start/. $REPOSITORY_ROOT/workshop/
    ```

    ```powershell
    # PowerShell
    New-Item -Type Directory -Path $REPOSITORY_ROOT/workshop -Force && `
        Copy-Item -Path $REPOSITORY_ROOT/save-points/step-01/start/* -Destination $REPOSITORY_ROOT/workshop -Recurse -Force
    ```

## Setting Up LLM Access

In the previous [00: Development Environment Setup](./00-setup.md), we created a PAT for GitHub Models access and an API key for Azure OpenAI instance access. Let's configure these for use in the application.

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Run the following command to save the previously generated values.

    ```bash
    # GitHub Models
    dotnet user-secrets --project ./MafWorkshop.Agent set GitHub:Token $githubToken
    ```

   Run the following only if you have an Azure subscription.

    ```bash
    # Azure OpenAI
    dotnet user-secrets --project ./MafWorkshop.Agent set Azure:OpenAI:Endpoint $endpoint
    dotnet user-secrets --project ./MafWorkshop.Agent set Azure:OpenAI:ApiKey $apiKey
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

1. Run the application.

    ```bash
    dotnet watch run --project ./MafWorkshop.Agent
    ```

1. Verify that the web browser opens automatically and shows a 404 error page.

   ![404 Error Page](./images/step-01-image-01.png)

   Since nothing has been added yet, a 404 error page should appear as expected.

1. Press `CTRL`+`C` in the terminal to stop the application execution.

## Connecting LLM

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

1. Open the `./MafWorkshop.Agent/Program.cs` file and find the comment `// ChatClientFactory 클래스 추가하기` and add the following content. The code below is a factory method pattern that finds the `LlmProvider` value from the `IConfiguration` instance, creates an `IChatClient` instance using GitHub Models connection information if the value is `GitHubModels`, and creates an `IChatClient` instance using Azure OpenAI connection information if it's `AzureOpenAI`.

    ```csharp
    // ChatClientFactory 클래스 추가하기
    public class ChatClientFactory
    {
        public static IChatClient CreateChatClient(IConfiguration config)
        {
            var provider = config["LlmProvider"] ?? throw new InvalidOperationException("Missing configuration: LlmProvider");
            IChatClient chatClient = provider switch
            {
                "GitHubModels" => CreateGitHubModelsChatClient(config),
                "AzureOpenAI" => CreateAzureOpenAIChatClient(config),
                _ => throw new NotSupportedException($"The specified LLM provider '{provider}' is not supported.")
            };
    
            return chatClient;
        }
    
        private static IChatClient CreateGitHubModelsChatClient(IConfiguration config)
        {
            var provider = config["LlmProvider"];
    
            var github = config.GetSection("GitHub");
            var endpoint = github["Endpoint"] ?? throw new InvalidOperationException("Missing configuration: GitHub:Endpoint");
            var token = github["Token"] ?? throw new InvalidOperationException("Missing configuration: GitHub:Token");
            var model = github["Model"] ?? throw new InvalidOperationException("Missing configuration: GitHub:Model");
    
            Console.WriteLine($"Using {provider}: {model}");
    
            var credential = new ApiKeyCredential(token);
            var options = new OpenAIClientOptions()
            {
                Endpoint = new Uri(endpoint)
            };
    
            var client = new OpenAIClient(credential, options);
            var chatClient = client.GetChatClient(model)
                                   .AsIChatClient();
    
            return chatClient;
        }
    
        private static IChatClient CreateAzureOpenAIChatClient(IConfiguration config)
        {
            var provider = config["LlmProvider"];
    
            var azure = config.GetSection("Azure:OpenAI");
            var endpoint = azure["Endpoint"] ?? throw new InvalidOperationException("Missing configuration: Azure:OpenAI:Endpoint");
            var apiKey = azure["ApiKey"] ?? throw new InvalidOperationException("Missing configuration: Azure:OpenAI:ApiKey");
            var deploymentName = azure["DeploymentName"] ?? throw new InvalidOperationException("Missing configuration: Azure:OpenAI:DeploymentName");
    
            Console.WriteLine($"Using {provider}: {deploymentName}");
    
            var credential = new ApiKeyCredential(apiKey);
            var options = new OpenAIClientOptions
            {
                Endpoint = new Uri($"{endpoint.TrimEnd('/')}/openai/v1/")
            };
    
            var client = new ResponsesClient(deploymentName, credential, options);
            var chatClient = client.AsIChatClient();
    
            return chatClient;
        }
    }
    ```

1. In the same file, find the comment `// IChatClient 인스턴스 생성하기` and enter the following. Use the factory method pattern created earlier to generate a GitHub Models or Azure OpenAI instance as `IChatClient` type.

    ```csharp
    // IChatClient 인스턴스 생성하기
    IChatClient? chatClient = ChatClientFactory.CreateChatClient(builder.Configuration);
    ```

1. In the same file, find the comment `// IChatClient 인스턴스 등록하기` and enter the following. Register the `IChatClient` instance created earlier as a dependency object.

    ```csharp
    // IChatClient 인스턴스 등록하기
    builder.Services.AddChatClient(chatClient);
    ```

## Creating a Single Agent

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Open the `./MafWorkshop.Agent/Program.cs` file and find the comment `// Writer 에이전트 추가하기` and enter the following. Agents can be added in various ways, but here we use the simplest method by entering the agent name and persona/instructions.

    ```csharp
    // Writer 에이전트 추가하기
    builder.AddAIAgent(
        name: "writer",
        instructions: "You write short stories (300 words or less) about the specified topic."
    );
    ```

1. In the same file, find the comment `// OpenAI 관련 응답 히스토리 핸들러 등록하기` and enter the following. Directly register service instances that store the responses and conversation history generated by the agent as dependency objects without implementing separate logic.

    ```csharp
    // OpenAI 관련 응답 히스토리 핸들러 등록하기
    builder.Services.AddOpenAIResponses();
    builder.Services.AddOpenAIConversations();
    ```

1. In the same file, find the comment `// OpenAI 관련 응답 히스토리 미들웨어 설정하기` and enter the following. Add endpoints that invoke the responses and conversation history generated by the agent through middleware.

    ```csharp
    // OpenAI 관련 응답 히스토리 미들웨어 설정하기
    app.MapOpenAIResponses();
    app.MapOpenAIConversations();
    ```

## Adding Dev UI

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Open the `./MafWorkshop.Agent/Program.cs` file and find the comment `// Dev UI 미들웨어 설정하기` and enter the following. Add the `/devui` endpoint through middleware to load the Dev UI screen in the local development environment.

    ```csharp
    if (builder.Environment.IsDevelopment() == false)
    {
        app.UseHttpsRedirection();
    }
    // Dev UI 미들웨어 설정하기
    else
    {
        app.MapDevUI();
    }
    ```

## Running the Single Agent

1. Verify that you are in the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Run the application.

    ```bash
    dotnet run --project ./MafWorkshop.Agent
    ```

1. Verify that the terminal shows a message indicating GitHub Models is currently connected.

    ```text
    Using GitHubModels: openai/gpt-5-mini
    ```

1. Press `CTRL`+`C` in the terminal to stop the application.

1. **If you have an Azure subscription**, open the `./MafWorkshop.Agent/appsettings.json` file and change the `LlmProvider` value to `AzureOpenAI` as follows.

    ```jsonc
    {
      // 변경 전
      "LlmProvider": "GitHubModels",
    
      // 변경 후
      "LlmProvider": "AzureOpenAI",
    }
    ```

1. Run the application.

    ```bash
    dotnet run --project ./MafWorkshop.Agent
    ```

1. Verify that the terminal shows a message indicating Azure OpenAI is currently connected.

    ```text
    Using AzureOpenAI: gpt-5-mini
    ```

1. Press `CTRL`+`C` in the terminal to stop the application.

1. Run the application again.

    ```bash
    dotnet watch run --project ./MafWorkshop.Agent
    ```

1. Verify that the web browser opens automatically and shows the DevUI page.

   ![DevUI Page - Single Agent](./images/step-01-image-02.png)

   Send a message and check the results.

   ![Writer Agent Execution Result](./images/step-01-image-03.png)

1. Press `CTRL`+`C` in the terminal to stop the application execution.

## Verifying the Complete Result

The completed version of this session can be found at `$REPOSITORY_ROOT/save-points/step-01/complete`.

1. If you have the `workshop` directory from the previous exercise, delete it or rename it. For example: `workshop-step-01`
1. Open a terminal and run the following commands in order to create the workshop directory and copy the starting project.

    ```bash
    # zsh/bash
    mkdir -p $REPOSITORY_ROOT/workshop && \
        cp -a $REPOSITORY_ROOT/save-points/step-01/complete/. $REPOSITORY_ROOT/workshop/
    ```

    ```powershell
    # PowerShell
    New-Item -Type Directory -Path $REPOSITORY_ROOT/workshop -Force && `
        Copy-Item -Path $REPOSITORY_ROOT/save-points/step-01/complete/* -Destination $REPOSITORY_ROOT/workshop -Recurse -Force
    ```

1. Move to the workshop directory.

    ```bash
    cd $REPOSITORY_ROOT/workshop
    ```

1. Follow the previous [Setting Up LLM Access](#setting-up-llm-access) section to configure LLM access.
1. Build the entire project.

    ```bash
    dotnet restore && dotnet build
    ```

1. Follow the [Running the Single Agent](#running-the-single-agent) section.

---

Congratulations! You have completed developing a single agent backend using Microsoft Agent Framework. Now proceed to the next step!

👈 [00: Development Environment Setup](./00-setup.md) | [02: Integrating Frontend UI with Microsoft Agent Framework](./02-ui-integration-with-maf.md) 👉

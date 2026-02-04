# Microsoft Agent Framework Workshop in a Day

Workshop materials for integrating [Microsoft Agent Framework (MAF)](https://aka.ms/agent-framework) with [Copilot Studio](https://learn.microsoft.com/microsoft-copilot-studio/).

> 📝 **Note**: This is an English translation of the original Korean workshop materials available at [Azure-Samples/maf-workshop-in-a-day-ko](https://github.com/Azure-Samples/maf-workshop-in-a-day-ko)

![Hero Image](./assets/hero.jpg)

## Prerequisites

This workshop is primarily designed to run on [GitHub Codespaces](https://docs.github.com/codespaces), so you only need to prepare the following:

- A Chromium-based web browser ([Microsoft Edge](https://microsoft.com/edge), [Google Chrome](http://chrome.google.com), etc.)
- [Azure subscription](https://azure.microsoft.com/free)
- [GitHub personal account (free)](http://github.com/signup) 👉 If using a work account, it may malfunction according to company policies, so please use a personal account.
- [Microsoft Copilot Studio trial subscription](https://go.microsoft.com/fwlink/?LinkId=2107702)

## Overall Architecture

Upon completing this workshop, you will have built the following system:

![Overall System Architecture](./assets/architecture.png)

## Getting Started

This workshop consists of the following content. All sessions are designed for self-paced learning, so feel free to continue working through them even if you don't finish during the workshop time.

| Session                                                                  | Link                                                                                                        |
|--------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------|
| 00: Development Environment Setup                                         | [00-setup.md](./docs/00-setup.md)                                                                           |
| 01: Building a Single Agent with Microsoft Agent Framework               | [01-single-agent-with-maf.md](./docs/01-single-agent-with-maf.md)                                           |
| 02: Integrating Frontend UI with Microsoft Agent Framework               | [02-ui-integration-with-maf.md](./docs/02-ui-integration-with-maf.md)                                       |
| 03: Building Multi-Agent with Microsoft Agent Framework                  | [03-multi-agent-with-maf.md](./docs/03-multi-agent-with-maf.md)                                             |
| 04: Orchestrating Frontend Web UI and Backend Agent with Aspire          | [04-aspire-orchestration.md](./docs/04-aspire-orchestration.md)                                             |
| 05: Developing an MCP Server                                             | [05-mcp-server-development.md](./docs/05-mcp-server-development.md)                                         |
| 06: Integrating MCP Server with Microsoft Agent Framework                | [06-mcp-server-integration-with-maf.md](./docs/06-mcp-server-integration-with-maf.md)                       |
| 07: Building Agent and Integrating MCP Server in Copilot Studio **(Optional)** | [07-mcp-server-integration-with-copilot-studio.md](./docs/07-mcp-server-integration-with-copilot-studio.md) |

## Additional Resources

- [Microsoft Agent Framework (MAF)](https://aka.ms/agent-framework)
- [Copilot Studio](https://learn.microsoft.com/microsoft-copilot-studio/)
- [Model Context Protocol (MCP)](https://modelcontextprotocol.io)
- [MCP .NET Samples](https://aka.ms/mcp-dotnet-samples)

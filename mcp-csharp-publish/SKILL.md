---
name: mcp-csharp-publish
description: Guide for publishing and deploying MCP servers built with the C# SDK. Covers NuGet packaging for stdio servers, Docker containerization for HTTP servers, and Azure deployment options.
license: Complete terms in LICENSE.txt
---

# MCP C# Server Publishing Guide

## Overview

Publish and deploy your C# MCP server. This skill covers NuGet packaging (for stdio servers), Docker containerization (for HTTP servers), and Azure cloud deployment.

---

# Process

## 📦 Publishing Destinations

| Transport | Primary Destination | Secondary Options |
|-----------|--------------------|--------------------|
| **stdio** | NuGet.org | Local folder, private NuGet feed |
| **HTTP** | Docker container | Azure Container Apps, App Service |

---

## 🔧 NuGet Publishing (stdio Servers)

NuGet is the recommended distribution method for stdio MCP servers. Users can run your server using the `dnx` tool runner.

### 1. Configure Package Properties

Update your `.csproj` file:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    
    <!-- Package configuration -->
    <PackAsTool>true</PackAsTool>
    <ToolCommandName>mymcpserver</ToolCommandName>
    <PackageId>YourUsername.MyMcpServer</PackageId>
    <Version>1.0.0</Version>
    <Authors>Your Name</Authors>
    <Description>MCP server for interacting with MyService API</Description>
    <PackageProjectUrl>https://github.com/yourusername/mymcpserver</PackageProjectUrl>
    <RepositoryUrl>https://github.com/yourusername/mymcpserver</RepositoryUrl>
    <PackageLicenseExpression>MIT</PackageLicenseExpression>
    <PackageTags>mcp;modelcontextprotocol;ai;llm</PackageTags>
    <PackageReadmeFile>README.md</PackageReadmeFile>
    
    <!-- Multi-platform support -->
    <RuntimeIdentifiers>win-x64;linux-x64;osx-x64;osx-arm64</RuntimeIdentifiers>
  </PropertyGroup>

  <ItemGroup>
    <None Include="README.md" Pack="true" PackagePath="\" />
  </ItemGroup>
</Project>
```

### 2. Configure server.json

Create `.mcp/server.json` for MCP Registry metadata:

```json
{
  "$schema": "https://static.modelcontextprotocol.io/schemas/2025-10-17/server.schema.json",
  "description": "MCP server for interacting with MyService API",
  "name": "io.github.yourusername/mymcpserver",
  "version": "1.0.0",
  "packages": [
    {
      "registryType": "nuget",
      "registryBaseUrl": "https://api.nuget.org",
      "identifier": "YourUsername.MyMcpServer",
      "version": "1.0.0",
      "transport": {
        "type": "stdio"
      },
      "packageArguments": [],
      "environmentVariables": [
        {
          "name": "API_KEY",
          "value": "{api_key}",
          "variables": {
            "api_key": {
              "description": "API key for MyService authentication",
              "isRequired": true,
              "isSecret": true
            }
          }
        }
      ]
    }
  ],
  "repository": {
    "url": "https://github.com/yourusername/mymcpserver",
    "source": "github"
  }
}
```

### 3. Build and Pack

```bash
# Build the project
dotnet build -c Release

# Create the NuGet package
dotnet pack -c Release

# Packages are created in bin/Release/
ls bin/Release/*.nupkg
```

### 4. Publish to NuGet.org

```bash
# Push to NuGet.org
dotnet nuget push bin/Release/*.nupkg \
  --api-key YOUR_NUGET_API_KEY \
  --source https://api.nuget.org/v3/index.json

# Or push to test environment first
dotnet nuget push bin/Release/*.nupkg \
  --api-key YOUR_NUGET_API_KEY \
  --source https://apiint.nugettest.org/v3/index.json
```

### 5. Users Install Your Server

After publishing, users configure their MCP client:

```json
{
  "servers": {
    "MyMcpServer": {
      "type": "stdio",
      "command": "dnx",
      "args": ["YourUsername.MyMcpServer@1.0.0", "--yes"],
      "env": {
        "API_KEY": "${input:api_key}"
      }
    }
  }
}
```

**Load [📋 NuGet Publishing Guide](./reference/nuget_publishing.md) for detailed instructions.**

---

## 🐳 Docker Containerization (HTTP Servers)

Docker is the recommended distribution method for HTTP MCP servers.

### 1. Create Dockerfile

```dockerfile
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src

# Copy and restore
COPY *.csproj ./
RUN dotnet restore

# Copy and build
COPY . ./
RUN dotnet publish -c Release -o /app

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:10.0
WORKDIR /app
COPY --from=build /app .

# Configure the server
ENV ASPNETCORE_URLS=http://+:8080
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8080/health || exit 1

ENTRYPOINT ["dotnet", "MyMcpServer.dll"]
```

### 2. Build and Run Locally

```bash
# Build the image
docker build -t mymcpserver:latest .

# Run locally
docker run -d \
  -p 3001:8080 \
  -e API_KEY=your-api-key \
  --name mymcpserver \
  mymcpserver:latest

# Test the server
curl http://localhost:3001/health
```

### 3. Push to Container Registry

```bash
# Tag for Docker Hub
docker tag mymcpserver:latest yourusername/mymcpserver:1.0.0
docker push yourusername/mymcpserver:1.0.0

# Or Azure Container Registry
az acr login --name yourregistry
docker tag mymcpserver:latest yourregistry.azurecr.io/mymcpserver:1.0.0
docker push yourregistry.azurecr.io/mymcpserver:1.0.0
```

**Load [📋 Docker Deployment Guide](./reference/docker_deployment.md) for production configurations.**

---

## ☁️ Azure Deployment

### Azure Container Apps (Recommended)

Best for: Serverless HTTP MCP servers with auto-scaling.

```bash
# Create Container App
az containerapp create \
  --name mymcpserver \
  --resource-group mygroup \
  --environment myenvironment \
  --image yourregistry.azurecr.io/mymcpserver:1.0.0 \
  --target-port 8080 \
  --ingress external \
  --min-replicas 0 \
  --max-replicas 10 \
  --secrets api-key=secretref:api-key \
  --env-vars API_KEY=secretref:api-key
```

### Azure App Service

Best for: Traditional web hosting with more control.

```bash
# Create App Service
az webapp create \
  --name mymcpserver \
  --resource-group mygroup \
  --plan myplan \
  --deployment-container-image-name yourregistry.azurecr.io/mymcpserver:1.0.0

# Configure environment
az webapp config appsettings set \
  --name mymcpserver \
  --resource-group mygroup \
  --settings API_KEY=@Microsoft.KeyVault(VaultName=myvault;SecretName=api-key)
```

**Load [📋 Azure Deployment Guide](./reference/azure_deployment.md) for complete instructions.**

---

## 🔐 Security Considerations

### Secrets Management

**Never commit secrets to source control!**

```csharp
// Good - read from environment
var apiKey = Environment.GetEnvironmentVariable("API_KEY")
    ?? throw new InvalidOperationException("API_KEY required");

// For Azure - use Key Vault
builder.Configuration.AddAzureKeyVault(
    new Uri($"https://{vaultName}.vault.azure.net/"),
    new DefaultAzureCredential());
```

### Production Checklist

- [ ] API keys stored in environment variables or Key Vault
- [ ] HTTPS enabled for HTTP transport
- [ ] Health check endpoint implemented
- [ ] Logging configured appropriately
- [ ] Rate limiting considered
- [ ] Input validation on all parameters

---

## 📋 Publishing Workflow

### For stdio (NuGet) Servers

```bash
# 1. Update version in .csproj
# 2. Build and test
dotnet build -c Release
dotnet test

# 3. Pack
dotnet pack -c Release

# 4. Publish
dotnet nuget push bin/Release/*.nupkg --source nuget.org

# 5. Verify on NuGet.org
# https://www.nuget.org/packages/YourUsername.MyMcpServer
```

### For HTTP (Docker) Servers

```bash
# 1. Update version in Dockerfile/compose
# 2. Build and test
docker build -t mymcpserver:1.0.0 .
docker run --rm mymcpserver:1.0.0 --help

# 3. Push to registry
docker push yourregistry/mymcpserver:1.0.0

# 4. Deploy to Azure
az containerapp update --name mymcpserver --image mymcpserver:1.0.0

# 5. Verify deployment
curl https://mymcpserver.azurecontainerapps.io/health
```

---

## Related Skills

- **[mcp-csharp-create](../mcp-csharp-create/SKILL.md)** - Creating your MCP server
- **[mcp-csharp-debug](../mcp-csharp-debug/SKILL.md)** - Running and debugging
- **[mcp-csharp-test](../mcp-csharp-test/SKILL.md)** - Testing and evaluation

---

# Reference Files

## 📚 Documentation Library

- [📋 NuGet Publishing Guide](./reference/nuget_publishing.md) - Complete NuGet packaging and publishing
- [📋 Docker Deployment Guide](./reference/docker_deployment.md) - Docker containerization best practices
- [📋 Azure Deployment Guide](./reference/azure_deployment.md) - Azure Container Apps and App Service

### External Documentation
- **NuGet Publishing**: https://learn.microsoft.com/nuget/nuget-org/publish-a-package
- **Azure Container Apps**: https://learn.microsoft.com/azure/container-apps/
- **Docker Best Practices**: https://docs.docker.com/develop/develop-images/dockerfile_best-practices/

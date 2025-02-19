# Agentic AI and MCP (Model Context Protocol) example with Tanzu Platform for Cloud Foundry

The following are steps on how to recreate and follow along a recent live demo by Corby Page and Kirti Apte on [Cloud Foundry Weekly: episode 45](https://www.youtube.com/watch?v=V-eybisoNII)

## Pre-reqs
- [IntelliJ IDEA](https://www.jetbrains.com/idea/download/) (Ultimate or Community Edition)
- [Java JDK 17](https://www.oracle.com/java/technologies/downloads/#java17) or later installed, added to path, and JAVA_HOME configured
- [Maven 3.9.9](https://maven.apache.org/download.cgi) or later installed, added to path, and MAVEN_HOME configured
- A running Cloud Foundry environment and the following details recorded (see guide here on [how to install Tanzu Platform for Cloud Foundry](https://github.com/KeithRichardLee/VMware-Tanzu-Guides/blob/main/Tanzu-AI-Solutions/Tanzu-AI-Solutions-getting-started-guide.md))
  - API endpoint
  - Username
  - Password
  - Org name
  - Space name

## Anthropic Claude
- Sign up for a [Claude account](https://claude.ai/) (free or paid plans)
- Download and install [Claude Desktop](https://claude.ai/download)

## JetBrains MCP Server
### Install JetBrains MCP Server plugin into IDE
- Download [JetBrains MCP Server plugin](https://plugins.jetbrains.com/plugin/26071-mcp-server)
- Install plugin
  - IntelliJ > IDE settings > Plugins > Install Plugin from Disk

### Add JetBrains MCP Server config to Claude Desktop config
- Claude Desktop > Settings > Developer > Edit Config > open `claude_desktop_config.json` in editor of choice
- Add the following configuration to the `claude_desktop_config.json` file
```
  {
    "mcpServers": {
      "jetbrains": {
        "command": "npx",
        "args": ["-y", "@jetbrains/mcp-proxy"]
      }
    }
  }
```



## Cloud Foundry MCP server
### Download and build the Cloud Foundry MCP server
```
git clone https://github.com/cpage-pivotal/cloud-foundry-mcp.git
./mvnw clean package
```
Take note of the path to the built jar in the mvnw output as we will need it in the next step

### Add Cloud Foundry MCP server config to Claude Desktop config
Claude Desktop > Settings > Developer > Edit Config > open `claude_desktop_config.json` in editor of choice and add the Cloud Foundry MCP server config to the existing config

```
{
  "mcpServers": {
    "cloud-foundry": {
      "command": "java",
      "args": [
        "-Dspring.ai.mcp.server.transport=stdio", "-Dlogging.file.name=cloud-foundry-mcp.log", "-jar" ,
        "/path/to/cloud-foundry-mcp/target/cloud-foundry-mcp-0.0.1-SNAPSHOT.jar",
        "--server.port=8040"
      ],
      "env": {
        "CF_APIHOST": "[Your CF API Endpoint e.g. api.sys.mycf.com]",
        "CF_USERNAME": "[Your CF User]",
        "CF_PASSWORD": "[Your CF Password]",
        "CF_ORG": "[Your Org]",
        "CF_SPACE": "[Your Space]"
      }
    }
  }
}
```

See below for an example where both JetBrains and Cloud Foundry MCP servers are added to the Claude Desktop configuration. 
  - Note: if using Windows, use double \\ for the path to the jar we built in the previous step.
```
{
  "mcpServers": {
    "jetbrains": {
      "command": "npx",
      "args": ["-y", "@jetbrains/mcp-proxy"]
    },
    "cloud-foundry": {
      "command": "java",
      "args": [
        "-Dspring.ai.mcp.server.transport=stdio", "-Dlogging.file.name=cloud-foundry-mcp.log", "-jar" ,
        "C:\\Users\\Administrator\\IdeaProjects\\cloud-foundry-mcp\\target\\cloud-foundry-mcp-0.0.1-SNAPSHOT.jar",
        "--server.port=8040"
      ],
      "env": {
        "CF_APIHOST": "api.sys.tanzu.lab",
        "CF_USERNAME": "admin",
        "CF_PASSWORD": "this-is-a-safe-password",
        "CF_ORG": "tanzu-demos-org",
        "CF_SPACE": "demos-space"
      }
    }
  }
}
```

## Add Cloud Foundry cert to keystore (optional)
If your Cloud Foundry is using self-signed certs, you will need to add the cert to the java keystore eg
```
keytool -importcert -file "C:\Users\Administrator\Downloads\tpcf.crt" -cacerts -alias tpcf -storepass changeit
```

## Restart Claude Desktop and verify the JetBrains and Cloud Foundry MCP servers are running
- Claude Desktop > Settings > Developer

![MCP servers running](/Tanzu-AI-Solutions/assets/claude_desktop_jetbrains_mcp_server.jpg)

## Download and open Spring-Metal in IntelliJ
```
git clone https://github.com/nkuhn-vmw/GenAI-for-TPCF-Samples.git
```

## Congratulations!!
Now you can follow along with Corby and Kirti on the [Cloud Foundry Weekly episode 45](https://www.youtube.com/watch?v=V-eybisoNII)!

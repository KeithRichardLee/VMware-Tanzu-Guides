# MCP (Model Context Protocol) example with Tanzu Platform for Cloud Foundry

The following are steps on how to recreate and follow along a recent live demo by Corby Page and Kirti Apte on [Cloud Foundry Weekly: episode 45](https://www.youtube.com/watch?v=V-eybisoNII)

## Pre-reqs
- Java JDK 17 or later installed, added to path, and JAVA_HOME configured
- Maven 3.9.9 or later installed, added to path, and MAVEN_HOME configured
- [Claude account](https://claude.ai/) (free or paid plans)
- Cloud Foundry environment and the following details recorded
  - API endpoint
  - Username
  - Password
  - Org
  - Space

## Downloads
- Claude Desktop
  - https://claude.ai/download
- Cloud Foundry MCP server
  - https://github.com/cpage-pivotal/cloud-foundry-mcp/ 
 
## Install Claude Desktop

## Download/clone and build Cloud Foundry MCP server
```
git clone https://github.com/cpage-pivotal/cloud-foundry-mcp.git
./mvnw clean package
```
Take note of the path to the built jar in the mvnw output as we will need it in the next step.

## Add Cloud Foundary mcp server config to Claude Desktop config
Edit `claude_desktop_config.json`
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

See below for an example. Note: if using Windows, use double \\ for the path to the jar we built in the previous step.
```
{
  "mcpServers": {
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

## Start Claude Desktop and verify the Cloud Foundry MCP server is running

![CF MCP server](/Tanzu-AI-Solutions/assets/claude_desktop_cf_mcp_server.jpg)

Now you can follow along with Corby and Kirte on the [Cloud Foundry Weekly episode](https://www.youtube.com/watch?v=V-eybisoNII)!

### todo: add jetbrains mcp server

# Demo Notes

## Create .NET Application

```sh
dotnet new webapi --use-controllers --name DemoApi -o .
```

```sh
dotnet new gitignore
```

## Build and Deploy YAML

```yaml
# Starter pipeline
# Start with a minimal pipeline that you can customize to build and deploy your code.
# Add steps that build, run tests, deploy, and more:
# https://aka.ms/yaml

trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:

- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    version: '9.x'
- task: NuGetToolInstaller@1

- task: NuGetCommand@2
  inputs:
    command: 'restore'
    restoreSolution: '**/*.sln'
    feedsToUse: 'select'
  
- task: DotNetCoreCLI@2
  inputs:
    command: 'build'

- task: DotNetCoreCLI@2
  inputs:
    azureSubscription: 'Azure DevOps Person Pay as You Go(2d24be35-592e-4737-830e-0e3ac22456aa)'
    command: 'publish'
    publishWebProjects: true

- task: AzureRmWebAppDeployment@4
  inputs:
    ConnectionType: 'AzureRM'
    azureSubscription: 'Azure DevOps Person Pay as You Go(2d24be35-592e-4737-830e-0e3ac22456aa)'
    appType: 'webAppLinux'
    WebAppName: 'demo-api-live'
    packageForLinux: '$(System.DefaultWorkingDirectory)/**/*.zip'
```

## KeyVault AppSettings

```text
@Microsoft.KeyVault(SecretUri=https://demo-api-scs.vault.azure.net/secrets/SOMEVAR)
```

## Controller Endpoint

```csharp
[HttpGet("get-some-var")]
public IActionResult GetSomeVar()
{
    var someVar = Environment.GetEnvironmentVariable("SOME_VAR");
    if (string.IsNullOrEmpty(someVar))
    {
        return NotFound("Environment variable 'SOME_VAR' not found.");
    }
    return Ok(someVar);
}
```

## GitHub Advanced Security

```yaml
trigger:
  - main

pool:
  # Additional hosted image options are available: https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted#software
  vmImage: ubuntu-latest

steps:

  - task: AdvancedSecurity-Codeql-Init@1
    inputs:
      languages: "csharp"
      # Supported languages: csharp, cpp, go, java, javascript, python, ruby, swift
      # You can customize the initialize task: https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/advanced-security-codeql-init-v1?view=azure-pipelines
      # If you're using a self-hosted agent to run CodeQL, use `enableAutomaticCodeQLInstall` to automatically use the latest CodeQL bits on your agent:
      enableAutomaticCodeQLInstall: true

  #   Add your custom build steps here
  # - Ensure that all code to be scanned is compiled (often using a `clean` command to ensure you're building from a clean state).
  # - Disable the use of any build caching mechanisms as this can interfere with CodeQL's ability to capture all the necessary data during the build.
  # - Disable the use of any distributed/multithreaded/incremental builds as CodeQL needs to monitor executions of the compiler to construct an accurate representation of the application.
  # - For dependency scanning, ensure you have a package restore step for more accurate results.
  - task: UseDotNet@2
    inputs:
      packageType: 'sdk'
      version: '9.x'
  - task: NuGetToolInstaller@1

  - task: NuGetCommand@2
    inputs:
      command: 'restore'
      restoreSolution: '**/*.sln'
      feedsToUse: 'select'
    
  - task: DotNetCoreCLI@2
    inputs:
      command: 'build'

  - task: AdvancedSecurity-Dependency-Scanning@1 # More details on this task: https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/advanced-security-dependency-scanning-v1?view=azure-pipelines

  - task: AdvancedSecurity-Codeql-Analyze@1 # More details on this task: https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/advanced-security-codeql-analyze-v1?view=azure-pipelines
```

## .NET App Bad Code

```csharp
void GetUserData(string username)
{
    var connectionString = "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=SuperSecretPassword123!;";
    string query = "SELECT * FROM Users WHERE Username = '" + username + "'";

    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        SqlCommand command = new SqlCommand(query, connection);
        connection.Open();
        SqlDataReader reader = command.ExecuteReader();
    }
}

async void GetTopSellingProducts(string[] args)
{
    var client = new ChatClient("gpt-4o",  "API_KEY");

    ChatCompletion completion = client.CompleteChat("Say 'this is a test.'");

    Console.WriteLine($"[ASSISTANT]: {completion.Content[0].Text}");
}
```

## Nuget Packages

```sh
dotnet add package Newtonsoft.Json --version 12.0.3
```

```sh
dotnet add package OpenAI --version 2.1.0
```

## Test WAF

```sh
curl -X GET "https://bookfrontdoor-c2dcfsg5d0behxhe.z02.azurefd.net/test" \
  -H "User-Agent: sqlmap/1.0" \
  -H "Content-Type: application/json" \
  -H "X-Forwarded-For: 123.45.67.89" \
  -H "X-Original-URL: /../../../../etc/passwd"
```

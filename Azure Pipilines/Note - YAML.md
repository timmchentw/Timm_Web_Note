# YAML Pipeline

## Container
### Docker compose push ( .Net Core Web App)

```YAML
# ASP.NET Core (.NET Framework)
# Build and test ASP.NET Core projects targeting the full .NET Framework.
# Add steps that publish symbols, save build artifacts, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/dotnet-core

trigger:
- master

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: My_Azure_DevOps_Pipelines_Library
  - name: solution
    value: '**/*.sln'
  - name: buildPlatform
    value: 'Any CPU'
  - name: buildConfiguration
    value: 'Release'
  - name: AzureSubscription
    value: '...'
  - name: AzureContainerRegistry
    value: '...'
  - name: DockerComposeFile
    value: 'docker-compose.production.yml'

steps:

- task: DockerCompose@0
  inputs:
    containerregistrytype: 'Azure Container Registry'
    dockerComposeFile: '$(DockerComposeFile)'
    action: 'Build services'
    additionalImageTags: '$(Build.BuildNumber)'

- task: DockerCompose@0
  inputs:
    containerregistrytype: 'Azure Container Registry'
    azureSubscription: '$(AzureSubscription)'
    azureContainerRegistry: '$(AzureContainerRegistry)'
    dockerComposeFile: '$(DockerComposeFile)'
    action: 'Push services'
    additionalImageTags: '$(Build.BuildNumber)'

# Push compose config for release pipelines (Azure Web App doesn't support "merged docker-compose.yml" file)
# Reference: https://mjc.si/2020/10/08/modify-yml-files-with-powershell/
- task: PowerShell@2
  inputs:
    targetType: 'inline'
    script: |
      # Install and import the `powershell-yaml` module
      # Install module has a -Force -Verbose -Scope CurrentUser arguments which might be necessary in your CI/CD environment to install the module
      Install-Module -Name powershell-yaml -Force -Verbose -Scope CurrentUser
      Import-Module powershell-yaml
       
      # LoadYml function that will read YML file and deserialize it
      function LoadYml {
          param (
              $FileName
          )
      	# Load file content to a string array containing all YML file lines
          [string[]]$fileContent = Get-Content $FileName
          $content = ''
          # Convert a string array to a string
          foreach ($line in $fileContent) { $content = $content + "`n" + $line }
          # Deserialize a string to the PowerShell object
          $yml = ConvertFrom-YAML $content
          # return the object
          Write-Output $yml
      }
       
      # WriteYml function that writes the YML content to a file
      function WriteYml {
          param (
              $FileName,
              $Content
          )
      	#Serialize a PowerShell object to string
          $result = ConvertTo-YAML $Content
          #write to a file
          Set-Content -Path $FileName -Value $result
      }
       
      # Loading yml, setting new values and writing it back to disk
      $yml = LoadYml "$(Build.SourcesDirectory)/$(DockerComposeFile)"
      $yml.services.web.image = $yml.services.web.image + ":" + $(Build.BuildNumber)
      WriteYml "$(Build.StagingDirectory)/docker-compose.yml" $yml


- task: PublishPipelineArtifact@1
  inputs:
    targetPath: '$(Build.StagingDirectory)/docker-compose.yml'
    publishLocation: 'pipeline'
# You can pull this file in release pipeline for deployment
```

### Push to Azure Web App

```YAML
# You can move to Release pipeline (CD)
- task: AzureWebAppContainer@1
  inputs:
    azureSubscription: '$(AzureSubscription)'
    appName: 'MY_AZURE_WEB_APP'
    containers: 'your-ACR-name.azurecr.io/your_container_image_name:$(Build.BuildNumber)'
    multicontainerConfigFile: '$(Build.StagingDirectory)/docker-compose.yml'
```


## Traditional
### .Net Core build & push

```YAML
- task: NuGetToolInstaller@1

- task: NuGetCommand@2
  inputs:
    restoreSolution: '$(solution)'

- task: VSBuild@1
  inputs:
    solution: '$(solution)'
    msbuildArgs: '/p:DeployOnBuild=true /p:WebPublishMethod=Package /p:PackageAsSingleFile=true /p:SkipInvalidConfigurations=true /p:DesktopBuildPackageLocation="$(build.artifactStagingDirectory)\WebApp.zip" /p:DeployIisAppPath="Default Web Site"'
    platform: '$(buildPlatform)'
    configuration: '$(buildConfiguration)'

- task: VSTest@2
  inputs:
    platform: '$(buildPlatform)'
    configuration: '$(buildConfiguration)'

- task: DotNetCoreCLI@2
  inputs:
    command: 'publish'
    publishWebProjects: false
    projects: 'your_website_projects'
    arguments: '--output $(Build.ArtifactStagingDirectory)'

- task: PublishPipelineArtifact@1
  inputs:
    targetPath: '$(Build.ArtifactStagingDirectory)'
    publishLocation: 'pipeline'
```

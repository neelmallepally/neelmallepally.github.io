---
layout: post
title: "Why Azure DevOps Pipelines Copy Files Task is Not Copying Files"
date: 2021-04-04 
categories: azure devops
---
**Problem**

This is a weird issue that I encountered while setting azure devops pipeline using YAML. The pipeline has few tasks. The first task is to install `Bicep` on the azure dev ops agent and the run the `bicep build` command. The build command will take bicep file and generate a json arm template file. The next task is `CopyFiles@2` to copy the json file to build artifacts directory.

The first task `install & build` is working fine and generating the json file but the next task `Copy Files to Artifacts` is not copying the file to artifacts directory. Also, there are no errors shown in the pipeline logs. After little bit of searching and scratching head, the issue was with the path format.

Here is part of original YAML file
``` YAML
stages:
- stage: Build
  jobs:
    - job: Build
      displayName: Build and Validate
      pool:
        vmImage: 'ubuntu-latest'
      steps:
        - bash: | 
            curl -Lo bicep https://github.com/Azure/bicep/releases/latest/download/bicep-linux-x64
            chmod +x ./bicep
            ./bicep build $(System.DefaultWorkingDirectory)/$(BicepFilePath)
          displayName: 'Install Bicep'         
        
        - task: CopyFiles@2
          displayName: 'Copying bicep Json file to artifacts folder'
          inputs:
            Contents: '$(System.DefaultWorkingDirectory)\main.json'
            TargetFolder: '$(Build.artifactStagingDirectory)'
            OverWrite: true
            flattenFolders: true
        
        - task: PublishBuildArtifacts@1
          displayName: 'Publish Artifacts'
          inputs:
            PathtoPublish: '$(Build.ArtifactStagingDirectory)'
            ArtifactName: 'drop'
            publishLocation: 'Container'
```

**Solution**

Notice `pool - vmImage` in job build and file path for json file in inputs - Contents of `CopyFiles@2` task. For linux build agents the file path should be forward slash `/` not `\`.  



**References**

[https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/utility/copy-files?view=azure-devops&tabs=yaml](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/utility/copy-files?view=azure-devops&tabs=yaml)
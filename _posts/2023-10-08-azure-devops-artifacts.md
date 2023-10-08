---
title: "Downloading individual files from an Azure DevOps pipeline artifact using the REST API"
author: "Alex Oswald"
date: 2023-10-07
---

If you've come here, you know what you want. You want to be able to download individual files from an Azure DevOps pipeline artifact. The documentation makes no sense. So lets get straight to the point.

First, I want to point out that this only works with [PublishPipelineArtifact](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/publish-pipeline-artifact-v1), and **NOT** with [PublishBuildArtifacts](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/publish-build-artifacts-v1)

I've created a demo pipeline for this post. It simply creates two text files and publishes them as a pipeline artifact.

```yml
pool:
  vmImage: 'ubuntu-latest'

steps:
  - pwsh: |
      Write-Host "Test files"
      "Test string 1" | Out-File -FilePath .\test1.txt
      "Test string 2" | Out-File -FilePath .\test2.txt
    workingDirectory: $(Build.ArtifactStagingDirectory)

  - task: PublishPipelineArtifact@1
    inputs:
      ArtifactName: TestArtifact
      targetPath: $(Build.ArtifactStagingDirectory)
```

Let us start by hitting the [Artifacts - Get Artifact](https://learn.microsoft.com/en-us/rest/api/azure/devops/build/artifacts/get-artifact) so we can get some info about the artifact.

**Request**

```
https://dev.azure.com/oswaldtechnologies/add3132d-b1ce-4519-8299-4e67eecad1f5/_apis/build/builds/703/artifacts?artifactName=TestArtifact
```

**Response**

```json
{
    "id": 322,
    "name": "TestArtifact",
    "source": "12f1170f-54f2-53f3-20dd-22fc7dff55f9",
    "resource": {
        "type": "PipelineArtifact",
        "data": "973C2055701973A0FDFB695031EE3F9E7A91741016CA639E9D21ECCD1B387E9B01",
        "properties": {
            "RootId": "DC04A61FBB2E879A10EE8BA01B28B4623140546805A968AB2B491B2EE1BD2E4102",
            "artifactsize": "28",
            "HashType": "DEDUPNODEORCHUNK"
        },
        "url": "https://dev.azure.com/oswaldtechnologies/add3132d-b1ce-4519-8299-4e67eecad1f5/_apis/build/builds/703/artifacts?artifactName=TestArtifact&api-version=7.1",
        "downloadUrl": "https://artprodeus21.artifacts.visualstudio.com/A3e090689-466b-408e-a12e-87c169eff347/add3132d-b1ce-4519-8299-4e67eecad1f5/_apis/artifact/cGlwZWxpbmVhcnRpZmFjdDovL29zd2FsZHRlY2hub2xvZ2llcy9wcm9qZWN0SWQvYWRkMzEzMmQtYjFjZS00NTE5LTgyOTktNGU2N2VlY2FkMWY1L2J1aWxkSWQvNzAzL2FydGlmYWN0TmFtZS9UZXN0QXJ0aWZhY3Q1/content?format=zip"
    }
}
```

Notice `resource.data`. This is actually the `fileId`. And this is what the documentation does not mention. We use the `fileId` of the actual artifact, to get the artifacts manifest contents. The manifest contains an array of all items in the artifact, including their file path and file id. So now lets hit [Aritfacts - Get File](https://learn.microsoft.com/en-us/rest/api/azure/devops/build/artifacts/get-file) with the artifacts file id to get the manifest. Oddly, you must add `fileId` **and** `fileName` to the query, even though what you set `fileName` to doesn't matter. For this example, I'm going to call it `manifest.json`, because well, that's what it is.

**Request**

```
https://dev.azure.com/oswaldtechnologies/add3132d-b1ce-4519-8299-4e67eecad1f5/_apis/build/builds/703/artifacts?artifactName=TestArtifact&fileId=973C2055701973A0FDFB695031EE3F9E7A91741016CA639E9D21ECCD1B387E9B01&fileName=manifest.json
```

**Response**

```json
{
    "manifestFormat": "1.1.0",
    "items": [
        {
            "path": "/test1.txt",
            "blob": {
                "id": "D21C967E56201F344B44EE00F537263C3503AEB13931F99754F9E78E14E6C90C01",
                "size": 14
            }
        },
        {
            "path": "/test2.txt",
            "blob": {
                "id": "FE2A48E456C37C7BAF1F86D724E2C2B30658AA1A899201D61E23CE59A333A63801",
                "size": 14
            }
        }
    ],
    "manifestReferences": []
}
```

Now we can hit [Aritfacts - Get File](https://learn.microsoft.com/en-us/rest/api/azure/devops/build/artifacts/get-file) once more. This time let us get the contents of `test1.txt`. Again, it does not matter what you set `fileName` too, as long as the `fileId` matches the `blob.id` of the file you want to download.

**Request**

```
https://dev.azure.com/oswaldtechnologies/add3132d-b1ce-4519-8299-4e67eecad1f5/_apis/build/builds/703/artifacts?artifactName=TestArtifact&fileId=D21C967E56201F344B44EE00F537263C3503AEB13931F99754F9E78E14E6C90C01&fileName=test1.txt
```

**Response**

```txt
Test string 1
```

And there you have it! Figuring out how to do this was important for a project I was working on. I needed to download a small json file from artifacts that were over 10GB in size. It would be silly to download the whole artifact just to extract a small json file from it.

Give credit where credit is due. I found the solution to this problem via this [GitHub issues comment](https://github.com/MicrosoftDocs/vsts-rest-api-specs/issues/381#issuecomment-1612877259).
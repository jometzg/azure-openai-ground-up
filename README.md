# Azure OpenAI from the ground up
## Introduction
This tutorial is intended to help your understanding of Azure OpenAI from the developer's perspective. How to programmatically interact with Azure OpenAI and how to run chat sessions against the the [Azure OpenAI REST API](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference).

## Azure OpenAI

## Concepts

## Tutorial Prerequisites
The rest of this tutorial is based around executing HTTP REST requests against Azure OpenAI. All of the other SDKs and frameworks that get used in other programming languages such as Python or C# .NET will use these REST APIs. In addition, higher-level frameworks such as [langchain](https://www.langchain.com/) or [Semantic Kernel](https://github.com/microsoft/semantic-kernel) are built ontop of these APIs.

In the [Azure Portal](https://portal.azure.com/) provision the resources below:
1. Create a resource group openai-groundup-rg
2. In this resource group create and instance of Azure OpenAI
3. Take a note of its endpoint and key
4. In this OpenAI resource, open up AI Studio and deploy the following models:
5.   gpt-35-turbo
6.   text-embedding-ada-002
7. For later, deploy a storage account, create a bl;ob container and upload the same PDF from this repo into the blob container
8. Create an Azure Search instance
9. Take a note of its endpoint and key

In addition to the above Azure resources, the tutorial will use Visual Studio code and a marketplace extension  [REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) that will allow you to execute HTTP REST requests in Visual Studio Code. Alternatively, you can use other techniques, but this tutorial will concentrate on REST Client.

The above should allow you to run all of the steps in this tutorial.

## Tutorial Steps
### Your first OpenAI chat call
Inside Visual Studio code, create a new file and save as "OpenAI-samples.http". This will then indicate to VS Code that the REST client can then use this file to execute HTTP REST requests.

In the blank file type:
```
POST https://YOUR_RESOURCE_NAME.openai.azure.com/openai/deployments/YOUR_DEPLOYMENT_NAME/completions?api-version=2024-02-01
Content-Type: application/json
api-key: YOUR_RESOURCE_KEY

{
    "prompt": "Once upon a time",
    "max_tokens": 10
}
```
Save and a "send Request" link should appear. Click the link.

You should get a response like below:
```
HTTP/1.1 200 OK
Cache-Control: no-cache, must-revalidate
Content-Length: 307
Content-Type: application/json
access-control-allow-origin: *
apim-request-id: 97431f0d-859c-48db-bc01-9b5f6df8ea3d
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
x-content-type-options: nosniff
x-ms-region: West Europe
x-ratelimit-remaining-requests: 29
x-ratelimit-remaining-tokens: 29960
x-accel-buffering: no
x-ms-rai-invoked: true
x-request-id: 7b0041e6-b032-4bbe-8cd0-31efb80be5c2
x-ms-client-request-id: 97431f0d-859c-48db-bc01-9b5f6df8ea3d
azureml-model-session: turbo-0301-b90147a4
Date: Tue, 09 Apr 2024 15:42:01 GMT
Connection: close

{
  "id": "cmpl-9C7thWlxje46HDdYv69Ous51AZucM",
  "object": "text_completion",
  "created": 1712677321,
  "model": "gpt-35-turbo",
  "choices": [
    {
      "text": ", there was a Bookseller who lived in Manchester",
      "index": 0,
      "finish_reason": "length",
      "logprobs": null
    }
  ],
  "usage": {
    "prompt_tokens": 4,
    "completion_tokens": 10,
    "total_tokens": 14
  }
}
```

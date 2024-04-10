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
### Task 1 - your first OpenAI call
Inside Visual Studio code, create a new file and save as "openai-samples.http". This will then indicate to VS Code that the REST client can then use this file to execute HTTP REST requests.

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
Replace the 3 placeholders with the values from your Azure OpenAI instance. Save and a "Send Request" link should appear. Click the link.

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
What can be seen from above is that it worked (there will be a different answer each time) and that it used 4 prompt tokens and 10 completion tokens.

Try varying the *max_tokens* to see how this impacts the response.

There are also a lot of potentially interesting HTTP headers which may be used to understand more about the service and its capacity.

### Task 2 - parameterising the REST call
The REST client has several ways in which calls to APIs can be parameterised. This is really useful for later calls. The simplest approach is to declare some variables, set their value and then use these variables in the REST call.

The REST client has the means of having multiple requests in one file. These can be separated by a comment line. Add below to the file after the first request
```
### a parameterised request to completions
@openaiendpoint=YOUR_RESOURCE_NAME
@openaichatmode=YOUR_DEPLOYMENT_NAME
@openaikey=YOUR_RESOURCE_KEY

POST https://{{openaiendpoint}}.openai.azure.com/openai/deployments/{{openaichatmodel}}/completions?api-version=2023-05-15
api-key: {{openaikey}}
Content-Type: application/json

{
    "prompt": "Once upon a time",
    "max_tokens": 10
}
```

The above should allow you to reuse the parameters on later requests. A better way for files that are going to be put into a GitHub repo is to use an environment variable with VS Code. This firstly keeps secrets out of the file, but it also allows named environments that may have different values. For example:
```
"rest-client.environmentVariables": {
    "$shared": {
        "version": "v1",
        "prodToken": "foo",
        "nonProdToken": "bar"
    },
    "local": {
        "version": "v2",
        "host": "localhost",
        "token": "{{$shared nonProdToken}}",
        "secretKey": "devSecret"
    },
    "production": {
        "host": "example.com",
        "token": "{{$shared prodToken}}",
        "secretKey" : "prodSecret"
    }
}
```
In the above, there are local and production for distinct values and $shared for common values across environments. In VS Code this gets stored in .vscode with the name *settings.json*. This is the preferred method for dealing with variables, especially secrets such as OpenAI keys.

### Task 3 - chat completions
In the first examples, we are providing a simple prompt which then gets completed.

Most OpenAI implementations want to do chat - that is one or more questions that form a chat. This allows the easy creation of starting with a general question and then later asking a another question about the response. This mimics the ways in which humans discover information from others - starting more general and then honing the next question based on the response.

The format of the HTTP requests for chat completions is slightly different. It has a slightly different URL and a different request format.

```
### first chat completion. Note the URL change and the request body change

POST https://{{openaiendpoint}}.openai.azure.com/openai/deployments/{{openaichatmodel}}/chat/completions?api-version=2023-05-15
api-key: {{openaikey}}
Content-Type: application/json

{
    "messages": [
        {
            "role": "user",
            "content": "Does Azure OpenAI support customer managed keys?"
        }
    
    ]
}
```
Executing this request should produce something like (removing the response headers to save space):
```
{
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "Yes, Azure OpenAI supports customer managed keys through Azure Key Vault. Customers can manage encryption keys for their OpenAI resources in Azure Key Vault and use them for data encryption at rest and in transit. By integrating Azure Key Vault with OpenAI, customers can ensure their data is secure and compliant with their organization’s security policies.",
        "role": "assistant"
      }
    }
  ],
  "created": 1712678922,
  "id": "chatcmpl-9C8JWAZBcrs5736QjPMGKZAKX4WLV",
  "model": "gpt-35-turbo",
  "object": "chat.completion",
  "system_fingerprint": null,
  "usage": {
    "completion_tokens": 65,
    "prompt_tokens": 17,
    "total_tokens": 82
  }
}
```
Note that the response format is also different.

### Task 4 system prompts
In most chat scenarios our customers may want to build, just presenting Azure OpenAI with the user's query or prompt may be a little too uncontrolled. It would be really useful if the chat could be directed to answer the question in specfic ways or to restrict how the chat will respond to a user's prompt.

This is a whole subject area called [prompt engineering](https://en.wikipedia.org/wiki/Prompt_engineering). All of this us used to create a *system prompt* that gets sent to the API alongdise the actual user's prompt.

There is much work out there to help craft system prompts for specific use cases. This is outside this tutorial, but there are many resources that can help craft system prompts.

At an API level, a system prompt is merely another section in the JSON body that is sent with the HTTP POST request. In general, system prompts are added by the application sending the requests to the OpenAI REST API and not by the user. In this way, the system prompt can protect both the organisation and the user from prompts that doi not succeed in correctly answering a user query. Try this sample below:

```
### chat completion - but restrict output to Azure only conversations!!
POST https://{{openaiendpoint}}.openai.azure.com/openai/deployments/{{openaichatmodel}}/chat/completions?api-version=2023-05-15
api-key: {{openaikey}}
Content-Type: application/json

{
    "messages": [
        {
            "role": "system",
            "content": "You are an Azure-only assistant. Only give answers from Azure documentation. If the customer asks about another cloud provider, politely decline and respond that you are an Azure assistant."
        },
        {
            "role": "user",
            "content": "In AWS what is the container service called?"
        }
    
    ]
}
```

This can produce the following response:
```
{
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "I'm sorry, but as an Azure assistant, I'm not able to answer questions about AWS. However, if you have any questions about Azure Container Service or any other Azure-related topic, feel free to ask!",
        "role": "assistant"
      }
    }
  ],
  "created": 1712679694,
  "id": "chatcmpl-9C8VyVffqWzzUdDeGEp1L6IHK9Mim",
  "model": "gpt-35-turbo",
  "object": "chat.completion",
  "system_fingerprint": null,
  "usage": {
    "completion_tokens": 43,
    "prompt_tokens": 56,
    "total_tokens": 99
  }
}
```

In general, it can take much testing and refinement to end up with an effective system prompt. Projects that use Azure OpenAI chat should set some time aside for prompt testing and refinement - this should be done with business or product owner stakeholders to make sure it matches their expectations.

Prompt engineering is one of the main ways in which a user of the chat application can be stopped from subverting the intention of the chat application. This is know as [jailbreaking](https://learnprompting.org/docs/prompt_hacking/jailbreaking). End customer facing (as opposed to internally-facing) applications could be targets of jailbreaking attempts. Whilst most of these can be harmless, some of these may cause reputational damage to the organisation. 

### Task 5 - understanding more on context
Enter this query:
```


### chat completion - model trained date
POST https://{{openaiendpoint}}.openai.azure.com/openai/deployments/{{openaichatmodel}}/chat/completions?api-version=2023-05-15
api-key: {{openaikey}}
Content-Type: application/json

{
    "messages": [
        {
            "role": "user",
            "content": "Who is the reigning monarch of the United Kingdom?"
        }
    
    ]
}
```
If you ran this on a gpt-35 model, the answer is likely to be:
```
{
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "The reigning monarch of the United Kingdom is Queen Elizabeth II.",
        "role": "assistant"
      }
    }
  ],
  "created": 1712680378,
  "id": "chatcmpl-9C8h0DTTiC7SrJ5604yzwuMDeWdiw",
  "model": "gpt-35-turbo",
  "object": "chat.completion",
  "system_fingerprint": null,
  "usage": {
    "completion_tokens": 12,
    "prompt_tokens": 28,
    "total_tokens": 40
  }
}
```
gpt4 may give a more accurate answer.

This is to illustrate that the completion is based on 2 things. Firstly when the underlying model's training was performed and secondly on the prompt given. This is important to understand.

### Task 6 - conversations
As previously discussed, the Azure OpenAI API is stateless. That is it can only do a completion based upon the combination of its knowledge from its training and the prompt sent to it.

To mimic a conversation, each step of that conversation will add to the context sent to the API. At an API level, this is really simple, the JSON message sent to the API must add each user and *assistant* prompt as the user progresses through the conversation.

```
### chat completion - multiple prompt - initial question:
POST https://{{openaiendpoint}}.openai.azure.com/openai/deployments/{{openaichatmodel}}/chat/completions?api-version=2023-05-15
api-key: {{openaikey}}
Content-Type: application/json

{
    "messages": [
        {
            "role": "system",
            "content": "You are an Azure-only assistant. Only give answers from Azure documentation. If the customer asks about another cloud provider, suggest the Azure equivalent."
        },
        {
            "role": "user",
            "content": "can I assign a managed identity to an Azure mail enabled AD group?"
        }
    ]
}
```
If this gets the response:
```
{
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "No, you cannot assign a managed identity to an Azure mail-enabled AD group. A managed identity is a service principal, which represents an instance of an Azure service, and it should be used to authenticate and access Azure resources. On the other hand, an Azure mail-enabled AD group is used primarily to manage email addresses for a group, and it doesn't have authentication capabilities. However, you can assign a managed identity to an Azure resource group or an Azure virtual machine to provide authentication to access resources within that group or VM.",
        "role": "assistant"
      }
    }
  ],
  "created": 1712681035,
  "id": "chatcmpl-9C8rbRMOYFhRmBdEbZmCxrg9q7UEX",
  "model": "gpt-35-turbo",
  "object": "chat.completion",
  "system_fingerprint": null,
  "usage": {
    "completion_tokens": 105,
    "prompt_tokens": 55,
    "total_tokens": 160
  }
}
```
So, for a conversation, all we do is add an *assistant* section into the next request with the previous repsonse:

```
### chat completion -multiple prompts first answer and second question
POST https://{{openaiendpoint}}.openai.azure.com/openai/deployments/{{openaichatmodel}}/chat/completions?api-version=2023-05-15
api-key: {{openaikey}}
Content-Type: application/json

{
    "messages": [
        {
            "role": "system",
            "content": "You are an Azure-only assistant. Only give answers from Azure documentation. If the customer asks about another cloud provider, suggest the Azure equivalent."
        },
        {
            "role": "user",
            "content": "can I assign a managed identity to an Azure mail enabled AD group?"
        },
        {
            "role": "assistant",
            "content": "Yes, it is possible to assign a managed identity to an Azure mail-enabled AD group. This can be done through Azure Active Directory and the steps to do so are outlined in the Microsoft documentation: Assign a managed identity to an Azure AD Mail Enabled Security Group"
        },
        {
            "role": "user",
            "content": "How do I grant the managed identity permissions to the mail enabled AD group?"
        }   
    ]
}
```
This gives a response:
```
{
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "No, you cannot assign a managed identity to an Azure mail-enabled AD group. A managed identity is a service principal, which represents an instance of an Azure service, and it should be used to authenticate and access Azure resources. On the other hand, an Azure mail-enabled AD group is used primarily to manage email addresses for a group, and it doesn't have authentication capabilities. However, you can assign a managed identity to an Azure resource group or an Azure virtual machine to provide authentication to access resources within that group or VM.",
        "role": "assistant"
      }
    }
  ],
  "created": 1712681035,
  "id": "chatcmpl-9C8rbRMOYFhRmBdEbZmCxrg9q7UEX",
  "model": "gpt-35-turbo",
  "object": "chat.completion",
  "system_fingerprint": null,
  "usage": {
    "completion_tokens": 105,
    "prompt_tokens": 55,
    "total_tokens": 160
  }
}
```
In summary, conversations are just stateless API calls with the conversation history embedded in the prompt. Each of these items in the conversation history consumes tokens and so will be a factor in the performance and cost of the system. Ever increasing conversations can then hit a token limit. So control of conversation sessions is important for all applications.

### Task 7 Retrieval Augmented Generation
In previous tasks we have established that:
1. The Azure OpenAI APIs are stateless
2. The total context that the APIs have is therefore a combination of when the model was trained (its knowledge) and the prompt you give it.
3. There are limits to the amount of information that may be given in prompts - this is the number of tokens. So it is not possible to put a large number of documents into an OpenAI completion prompt.

[Retrieval Augmented Generation](https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview) (RAG) seeks to solve this problem by having a multi-step process to add extra grounding data to the prompt - without exceeeding prompt limits. This is implemented by having another service that works alongside Azure OpenAI to retrieve document fragments from previously indexed data. 

The most straightforward service for this is on Azure is [Azure AI Search](https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search) (used to be called Cognitive Search). Though it needs to be stated that RAG can use other mechanisms. But for now, this will concentrate on the simplest approach on Azure.

![alt text](./images/rag-architecture-diagram.png "RAG Overview")

In the diagram above, Azure AI Search is used to index data from one of a number of potential sources. One straightforward example is PDF documents stored in an Azure Blob Storage container. The *indexing* process is completely separate from the querying prompt process, so in order to use Azure AI Search in a RAG process, there must be a process that scans the blob storage container and indexes the result. This will be covered in later tasks, but what you end up with is a named index that may be used on the prompt.

The Azure OpenAI REST API already implements a standardised RAG process - so customers do not (initially at least) need to build this RAG process in code. This is documented under [completion extensions](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference#completions-extensions). The REST API call then defines the datasource for the indexed data as well as the actual user and system prompts.

```
curl -i -X POST YOUR_RESOURCE_NAME/openai/deployments/YOUR_DEPLOYMENT_NAME/extensions/chat/completions?api-version=2023-06-01-preview \
-H "Content-Type: application/json" \
-H "api-key: YOUR_API_KEY" \
-d \
'
{
    "temperature": 0,
    "max_tokens": 1000,
    "top_p": 1.0,
    "dataSources": [
        {
            "type": "AzureCognitiveSearch",
            "parameters": {
                "endpoint": "YOUR_AZURE_COGNITIVE_SEARCH_ENDPOINT",
                "key": "YOUR_AZURE_COGNITIVE_SEARCH_KEY",
                "indexName": "YOUR_AZURE_COGNITIVE_SEARCH_INDEX_NAME"
            }
        }
    ],
    "messages": [
        {
            "role": "user",
            "content": "What are the differences between Azure Machine Learning and Azure AI services?"
        }
    ]
}
'
```

# MTT CoHack Challenge : Troubleshooting CSDOT Azure Infrastructure and AI Solutions

## Introduction

Welcome to your new role as the **Azure Expert** for the **Contoso State Department of Transportation (CSDOT)**! Your primary responsibility involves troubleshooting an existing Azure environment that supports the department's serverless architecture. This setup includes Azure Functions, Event-based services, Key Vault, Storage, and App Services. Recently, the department has added Azure AI Compute Vision to its suite of tools. This AI service is designed to recognize vehicles on roads, a critical component for transport monitoring and billing systems. However, the system requires fine-tuning for optimal performance. As you settle into your role, you'll find that there are some **immediate challenges** for you to address: 

1. **Inter-service Communication Issues**: The Azure resources, crucial for the smooth operation of transport monitoring and billing systems, are experiencing communication problems. This disruption is affecting the overall functionality of the systems. 

2. **Azure AI Compute Vision Training**: Your expertise is needed to train the Azure AI Compute Vision service, specifically for vehicle recognition. This involves adjusting the model to accurately identify different types of vehicles on the roads, which is vital for effective monitoring and billing. 

Your role is *pivotal in ensuring that the Azure environment is stable, efficient, and capable of supporting the department's objectives*. 

Addressing these challenges will involve a mix of troubleshooting, system optimization, and leveraging your expertise in Azure services, particularly in AI and machine learning.

![CSDOT Azure Architecture](../images/CSDOT_architecture.jpg)

## Requirements

- Azure Subscription with OWNER permissions on  subscription level
- Pre-deploying the Co-Hack CSDOT Architecture through [MTTDemoDeploy](https://aka.ms/mttdemodeploy) (**Microsoft internal MTT team only access**) (deployment takes about 15min on average)
- Access to Azure AI Services (see [this Learn](https://learn.microsoft.com/en-us/legal/cognitive-services/openai/limited-access) article on how to get access)

## Learning Objectives

This hack will help you learn:

- Understand Serverless Azure Architecture
- Troubleshooting integration across different Azure Services
- Managing and interacting with Azure Key Vault and Key Vault permissions
- Using Application Insights for serverless architecture monitoring
- Using Azure AI Computer Vision 
- Creating and Training an Azure AI Computer Vision model

## Success Criteria

- Having a fully-workable Azure Serverless-based architecture working across all components (Functions, Event Grid, Azure Storage, Cosmos DB, Azure Key Vault,...)

- Having an Azure AI Computer Vision model configured and trained to recognize license plates in images

## The more detailed architecture

1.	The app starts with a web app, **https://<mttalias>csdotimageuploadapp.azurewebsites.net**, which simulates car traffic, generating 500 car images and store them in Azure Blob Storage.

> **Note:** You can run as many upload sequences as you want to validate the overall functionality of the Serverless Architecture

2.	**Event Grid** watches over the **Storage Account** activity, and when a new image file gets created in Blob, it triggers an **Azure Function "ProcessImage"**, which sends the image to **Azure AI – Computer Vision** – for text recognition.
3.	When the return-answer from Computer Vision is received by the Function, it sends metadata from the image (date, time, image file name, car license plate text) to a **CosmosDB database**, using another Azure Function **"SavePlateData"**.
4.	There is another Azure Function, which interacts with a Logic App to send monthly reports, but this is not really used as part of the architecture
5.	The **ProcessImage Function** is integrated with **Application Insights**, allowing for a Live View of the API-calls when generating car traffic from the web app in step 1. It also provides details about the integration across the different services from Application Maps, as well as more insights for Failures and Performance-alike metrics.
6.	Azure Functions is interacting with **Azure Key Vault**, to pick up all secrets such as storage account connection string, Computer Vision API Key, CosmosDB keys,… 

> **Note:** No changes are required - nor possible - to the Application Source Code used by the Functions, or the imageupload web app

> **Informational:** The following application settings are in use by the different components in the application architecture:

- AzureWebJobsStorage: Connection String to the Blob Storage Account
- computerVisionApiKey: Primary Key from Azure AI Computer Vision
- computerVisionAPIUrl: Endpoint URL from Azure AI Computer Vision
- cosmosDBAuthorizationKey: Primary Key from Azure Cosmos DB instance
- cosmosDBCollectionId: "Processed" is the name of the CosmosDB Collection
- cosmosDBDatabaseId: "LicensePlates" is the name of the CosmosDB database
- cosmosDBEndpointURL: The URL from the Cosmos DB instance
- datalakeConnection: ConnectionString to the Azure Blob Storage
- eventGridTopicEndPoint: URL from the Event Grid Topic resource
- eventGridTopicKey: Primary Key used to interact with Event Grid

### Challenge 1

After triggering an image upload process, images are arriving fine in Azure Blob Storage, but the Azure Function "ProcessImage" to send the image to Azure AI Computer Vision is not working as expected.

Your task is to troubleshoot and find the root cause of this problem. 

- Success Criteria 1: Azure Function is successfully picking up the images from Blob Storage, and sending them off to Azure AI Computer Vision.

### Challenge 2

After fixing the issue from Challenge 1, you will notice that still no license plate customer details are arriving in Cosmos DB. So something else is still wrong in the architecture.

Your task is to continue troubleshooting and find the root cause for this 2nd problem.

- Success Criteria 2: Azure Functions (SavePlateData) is successfully communicating with Cosmos DB instance to store license plate details when running an image upload sequence. 

### Challenge 3

While the Computer Vision API works fine, we want you to finetune it, by creating a model. The model should be based on generic car images with license plates (create a batch of 3-5 images from Bing Search), where you need to train it in such a way it primarily recognizes the license plate information in the image. However, you should train the model in such a way, it distincts between license plate info and 'other text' on the car image (e.g. brand, state, stickers,...)

- Success Criteria 3: Your coach will ask you to upload a car license plate image, and identify the different text objects in the image for accuracy.

### Challenge 4

With OCR text recognition capabilities in place, together with our custom trainer datamodel, we want to hand over similar functionality to our development team, to test this capability on their development environment. The scenario we are thinking of is using a Docker Image, which could run locally on a development workstation - or if time allows - you can host it in an Azure Container Instance. We heard from Microsoft there is a Microsoft-provided Docker image which provides computervision Read API functionality. 

Your task is to download and deploy this Docker image, using your Azure AI Computer Vision information. Once the container is running, you need to validate the functioning of the OCR Read API, by uploading an image file, and showing the JSON results. 

Success Criteria 4: You show your coach the JSON export file from within the running container workload URL

#### Additional Resources

- [App Service Key Vault References](https://learn.microsoft.com/en-us/azure/app-service/app-service-key-vault-references?tabs=azure-cli)
- [Key Vault Permissions](https://learn.microsoft.com/en-us/azure/key-vault/general/assign-access-policy?tabs=azure-portal)
- [Azure AI Vision](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/overview)
- [Azure AI OCR Read](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/quickstarts-sdk/client-library?tabs=linux%2Cvisual-studio&pivots=programming-language-csharp)
- [About OCR and IDP](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/overview-ocr)
- [Install Azure OCR Read Container](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/computer-vision-how-to-install-containers)
- [OCR Rest API Source](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2/operations/5d986960601faab4bf452005)




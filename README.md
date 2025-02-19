# Azure AI Services Containers

Using Azure AI services hosted in Azure enables application developers to focus on the infrastructure for their own code while benefiting from scalable services that are managed by Microsoft. However, in many scenarios, organizations require more control over their service infrastructure and the data that is passed between services.

Many of the Azure AI services APIs can be packaged and deployed in a *container*, enabling organizations to host Azure AI services in their own infrastructure; for example in local Docker servers, Azure Container Instances, or Azure Kubernetes Services clusters. Containerized Azure AI services need to communicate with an Azure-based Azure AI services account to support billing; but application data is not passed to the back-end service, and organizations have greater control over the deployment configuration of their containers, enabling custom solutions for authentication, scalability, and other considerations.

## Create a Resource Group for Azure AI Services containers

1. Open the Azure portal at `https://portal.azure.com`, and sign in using the Microsoft account associated with your Azure subscription.
2. In the top search bar, select **Resource group** and create a resource group with the following settings:
    - **Subscription**: *Visual Studio Premium with MSDN*
    - **Resource group name**: *rgAzureAIServicesContainers*
    - **Region**: *East US*
3. Review and create the resource group.

## Provision an Azure AI Services resource

If you don't already have one in your subscription, you'll need to provision an **Azure AI Services** resource.

1. Open the Azure portal at `https://portal.azure.com`, and sign in using the Microsoft account associated with your Azure subscription.
2. In the top search bar, search for *Azure AI services*, select **Azure AI services multi-service account** and create a resource with the following settings:
    - **Subscription**: *Visual Studio Premium with MSDN*
    - **Resource group**: *rgAzureAIServicesContainers*
    - **Region**: *East US*
    - **Name**: *resAzureAISvcs*
    - **Pricing tier**: Standard S0
3. Select the required checkboxes and create the resource.
4. Wait for deployment to complete, and then view the deployment details.
5. When the resource has been deployed, go to it and view its **Keys and Endpoint** page.
    - **Key**: *FJbIU9Nk1culgGU36iLVAmimfDxi2ZTTYNP7cmMxLr1kTx3Ce8ZaJQQJ99BBACYeBjFXJ3w3AAAEACOG0juy*
    - **Endpoint**: *https://resazureaisvcs.cognitiveservices.azure.com/*
These will need the endpoint and one of the keys from this page in the next procedure.

## Deploy and run a Sentiment Analysis container

Many commonly used Azure AI services APIs are available in container images. For a full list, check out the [Azure AI services documentation](https://learn.microsoft.com/en-us/azure/ai-services/cognitive-services-container-support#containers-in-azure-ai-services). In this exercise, we'll use the container image for the Text Analytics *Sentiment analysis* API; but the principles are the same for all of the available images.

1. In the Azure portal, on the **Home** page, select the **&#65291;Create a resource** button, search for *container instances*, and create a **Container Instances** resource with the following settings:

    - **Basics**:
        - **Subscription**: *Visual Studio Premium with MSDN*
        - **Resource group**: *rgAzureAIServicesContainers*
        - **Container name**: *cntsentimentanalysis*
        - **Region**: *Ease US*
        - **Availability zones**: None
        - **SKU**: Standard
        - **Image source**: Other Registry
        - **Image type**: Public
        - **Image**: `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:latest`
        - **OS type**: Linux
        - **Size**: 1 vcpu, 8 GB memory
    - **Networking**:
        - **Networking type**: Public
        - **DNS name label**: *Enter a unique name for the container endpoint*
        - **Ports**: *Change the TCP port from 80 to **5000***
    - **Advanced**:
        - **Restart policy**: On failure
        - **Environment variables**:

            | Mark as secure | Key | Value |
            | -------------- | --- | ----- |
            | Yes | `ApiKey` | *FJbIU9Nk1culgGU36iLVAmimfDxi2ZTTYNP7cmMxLr1kTx3Ce8ZaJQQJ99BBACYeBjFXJ3w3AAAEACOG0juy* |
            | Yes | `Billing` | *https://resazureaisvcs.cognitiveservices.azure.com/* |
            | No | `Eula` | `accept` |

        - **Command override**: [ ]
        - **Key management**: Microsoft-managed keys (MMK)
    - **Tags**:
        - *Don't add any tags*

2. Select **Review + create** then select **Create**. Wait for deployment to complete, and then go to the deployed resource.
    > **Note** Please note that deploying an Azure AI container to Azure Container Instances typically takes 5-10 minutes (provisioning) before they are ready to use.
3. Observe the following properties of your container instance resource on its **Overview** page:
    - **Status**: This should be *Running*.
    - **IP Address**: 20.231.246.136
    - **FQDN**: This is the *fully-qualified domain name* of the container instances resource, if we plan to use this to access the container instances instead of the IP address.

    > **Note**: We've deployed the Azure AI services container image for sentiment analysis to an Azure Container Instances (ACI) resource. You can use a similar approach to deploy it to a *[Docker](https://www.docker.com/products/docker-desktop)* host on your own computer or network by running the following command (on a single line) to deploy the sentiment analysis container to your local Docker instance, replacing *&lt;yourEndpoint&gt;* and *&lt;yourKey&gt;* with your endpoint URI and either of the keys for your Azure AI services resource.
    > The command will look for the image on your local machine, and if it doesn't find it there it will pull it from the *mcr.microsoft.com* image registry and deploy it to your Docker instance. When deployment is complete, the container will start and listen for incoming requests on port 5000.

    ```
    docker run --rm -it -p 5000:5000 --memory 8g --cpus 1 "mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:latest" Eula="accept" Billing="https://resazureaisvcs.cognitiveservices.azure.com/" ApiKey="FJbIU9Nk1culgGU36iLVAmimfDxi2ZTTYNP7cmMxLr1kTx3Ce8ZaJQQJ99BBACYeBjFXJ3w3AAAEACOG0juy"
    ```

## Use the container

1. Open a shell command prompt and enter the following **curl** command (shown below):.

    ```
    curl -X POST "http://20.231.246.136:5000/text/analytics/v3.1/sentiment" -H "Content-Type: application/json" --data-ascii "{'documents':[{'id':1,'text':'The performance was amazing! The sound could have been clearer.'},{'id':2,'text':'The food and service were unacceptable. While the host was nice, the waiter was rude and food was cold.'}]}"
    ```

2. Verify that the command returns a JSON document containing information about the sentiment detected in the two input documents (which should be postive and negative, in that order).

    > **Note**: You can also use SOAP UI and any other tool to test the REST endpoint.

## Clean Up

If you've finished experimenting with your container instance, you should delete it.

1. In the Azure portal, open the resource group where you created your resources for this exercise.
2. Select the container instance resource and delete it.

## Clean up resources

If you're not using the Azure resources created in this lab for other training modules, you can delete them to avoid incurring further charges.

1. Open the Azure portal at `https://portal.azure.com`, and in the top search bar, search for the resources you created in this lab.

2. On the resource page, select **Delete** and follow the instructions to delete the resource. Alternatively, you can delete the entire resource group to clean up all resources at the same time.

## More information

For more information about containerizing Azure AI services, see the [Azure AI Services containers documentation](https://learn.microsoft.com/azure/ai-services/cognitive-services-container-support).

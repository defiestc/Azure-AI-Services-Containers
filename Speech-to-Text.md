## Deploy and run a Speech to Text container

Many commonly used Azure AI services APIs are available in container images. For a full list, check out the [Azure AI services documentation](https://learn.microsoft.com/en-us/azure/ai-services/cognitive-services-container-support#containers-in-azure-ai-services). In this exercise, we'll use the container image for the Text Analytics *Sentiment analysis* API; but the principles are the same for all of the available images.

1. Go to the [Create](https://portal.azure.com/#create/Microsoft.ContainerInstances) page for Container Instances and create a **Container Instances** resource with the following settings:


    - **Basics**:
        - **Subscription**: *Visual Studio Premium with MSDN*
        - **Resource group**: *rgAzureAIServicesContainers*
        - **Container name**: *cntspeech2text*
        - **Region**: *East US*
        - **Availability zones**: None
        - **SKU**: Standard
        - **Image source**: Other Registry
        - **Image type**: Public
        - **Image**: `mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text:latest`
        - **OS type**: Linux
        - **Size**: 4 vcpu, 8 GB memory
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
    - **IP Address**: 20.253.46.177
    - **FQDN**: This is the *fully-qualified domain name* of the container instances resource, if we plan to use this to access the container instances instead of the IP address.


    > **Note**: We've deployed the Azure AI services container image for sentiment analysis to an Azure Container Instances (ACI) resource. You can use a similar approach to deploy it to a *[Docker](https://www.docker.com/products/docker-desktop)* host on your own computer or network by running the following command (on a single line) to deploy the sentiment analysis container to your local Docker instance, replacing *&lt;yourEndpoint&gt;* and *&lt;yourKey&gt;* with your endpoint URI and either of the keys for your Azure AI services resource.
    > The command will look for the image on your local machine, and if it doesn't find it there it will pull it from the *mcr.microsoft.com* image registry and deploy it to your Docker instance. When deployment is complete, the container will start and listen for incoming requests on port 5000.

    ```
    docker run --rm -it -p 5000:5000 --memory 8g --cpus 4 "mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text:latest" Eula="accept" Billing="https://resazureaisvcs.cognitiveservices.azure.com/" ApiKey="FJbIU9Nk1culgGU36iLVAmimfDxi2ZTTYNP7cmMxLr1kTx3Ce8ZaJQQJ99BBACYeBjFXJ3w3AAAEACOG0juy"
    ```

## Use the container

1. Open a shell command prompt and enter the following **curl** command (shown below).  This will submit a **png** file with images of **Tabs** and **Spaces**.
    ```
    curl -X POST "http://20.253.46.177:5000/vision/v3.2/read/syncAnalyze" -H "Content-Type: application/json" --data-ascii "{'url':'https://mystorageacct153963670.file.core.windows.net/myshare/myDirectory/tabs-vs-spaces.png?sv=2022-11-02&ss=bfqt&srt=o&sp=rwdlacupiytfx&se=2025-02-20T06:09:56Z&st=2025-02-19T22:09:56Z&spr=https,http&sig=zb4qZ5gge%2FQOgaP3E9y86JnfKGd3k9pE2lWB30QGMIU%3D'"
    ```

2. Verify that the command returns a JSON document containing information about the sentiment detected in the two input documents (which should be postive and negative, in that order).

    > **Note**: You can also use SOAP UI and any other tool to test the REST endpoint.

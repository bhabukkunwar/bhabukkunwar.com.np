---
date: 2024-09-06
title: Deploying a Node.js App Seamlessly with Azure Container Instances
description: I add aria2 as a download manager to a NixOS server to help bundle my Bandcamp downloads together
type: posts
tags:
  - Azure
  - Azure Container Instances
  - docker
  - node
---
Azure Container Instances is a Platform as a Service (PaaS) offering from Azure that allows you to run and deploy containers without the need of managing the underlying infrastructure.

Azure Container Registry, on the other hand, provides a secure, private registry for managing container images within the Azure ecosystem.

In this post, we will first learn how to containerize a Node.js application using Docker, and then deploy it to the cloud using Azure services such as Azure Container Instances (ACI) and Azure Container Registry (ACR).

Before we begin, make sure you have the following prerequisites installed on your machine:

* [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/)
    
    * [Docker](https://www.docker.com/products/docker-desktop/)
        
        * Git
            

### Step 1 : Containerizing a Node.js app using Docker

Let's start by cloning the following GitHub repository into your local machine.

[https://github.com/bhabukkunwar/AzureWeatherApp](https://github.com/bhabukkunwar/AzureWeatherApp)

It's a simple weather app written in Node.js. You'll find a *Dockerfile* in the root directory which will be used for building our Docker image. Here's the snippet of the docker file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725124369989/02150624-b219-43ed-8aff-abd6cfe6f6c1.png align="center")

In your root directory create an .env file. Here `app_id` is an API key provided by [Open Weather Map API](https://openweathermap.org/api). Go ahead an create an account to generate a free API key and use it in the .env file as below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725632166076/73605559-b3bc-47f6-a55b-81b8a96c68e4.png align="center")

Now to Dockerize the application execute the following steps inside project's root directory.

i. Build the Docker image using the following command.

```bash
docker build -t azure-weather-app:latest .
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725124620143/a95df3a4-ee43-4849-bb0d-05065e92eb4d.png align="center")

The `-t` flag allows us to tag our Docker images. The tag should come after the colon `:` in the format `image-name:tag`. Tagging is essential when pushing images to registries as it helps manage multiple versions of the same image. By default Docker tags all images as `latest` unless specified otherwise.

ii. Check whether the image was built correctly with the latest tag using *Docker images* command.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725124852606/299cbc12-8f66-4b4a-b278-de2169c16e93.png align="center")

iii. You can also test by running the container image locally with the following command:

```bash
docker run -p 80:80 azure-weather-app
```

The above command will deploy the app locally which can be accessed through *http://localhost* url in your browser.

### Step 2: Creating Azure Container Registry and pushing our Docker image

i. Inside your preferred terminal first login to your azure account using az cli.

```bash
az login
```

ii. Next, create a resource group. We will use this resource group to deploy our ACR and ACI.

```bash
az group create --name rg-az-weather-app --location eastus
```

iii. Create Azure Container Registry in the above resource group.

```bash
az acr create --resource-group rg-az-weather-app  --name azureweatherappregistry --sku Basic
```

iv. Now login should be possible to the newly created ACR using the following command.

```bash
az acr login --name azureweatherappregistry
```

v. In this section, we will create a service principal in Azure with the sole purpose of authenticating Azure Container Registry (ACR) with Docker and pushing our Docker image. It is strongly recommended to use a service principal for authenticating with ACR instead of using personal Azure account credentials.

**What is a Service Principal and why should you use it?**

A Service Principal (SP) in Azure acts as an identity used by applications or automated processes to access specific Azure resources. Instead of using your personal credentials (which could expose unnecessary access), the SP provides a way to control permissions more securely and granularly. In this guide, we’re using the SP to authenticate our Docker commands with Azure Container Registry (ACR), allowing us to push our container image securely.

Now that we've learned the importance of a Service Principal. Let's go ahead and create one.

* In your Azure Portal, go into App registrations and click on 'New registration'.
    
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725195095762/49e19d57-a53e-48b4-911d-b84435631011.png align="center")
            
            * Provide a user-facing name for our Service Principal(SP).
                
                    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725195155408/4c7dd6f8-2435-426b-bfb7-e9da5ae77513.png align="center")
                        
                        * Next, we need a client secret. Click on 'Certificates & secrets' and 'New client secret'.
                            
                                ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725195505558/9e2ff4e3-012b-4288-9db1-095e0edbef22.png align="center")
                                    
                                    * Set description and expiry date and click on 'Add'.
                                        
                                        * A client secret will be created. Make sure to copy the value, as it is only available once. We will use this secret later.
                                            
                                                ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725195725025/b45094f6-ef06-4968-88a2-2c824d12d393.png align="center")
                                                    
                                                    * Now, in the Access Control (IAM) for ACR, assign the 'AcrPush' role to our service principal following the steps below.
                                                        
                                                            * Inside 'azureweatherappregistry', click on Access control (IAM) and then 'Add role assignment'.
                                                                    
                                                                            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725197432897/771cabb1-d8c5-418a-b024-e764a5cbe16b.png align="center")
                                                                                    
                                                                                        * Search for 'AcrPush' and select it.
                                                                                                
                                                                                                        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725197404441/f9543df4-5bac-44df-b71e-e4faa0317b41.png align="center")
                                                                                                                
                                                                                                                    * In Members tab, make sure 'User, group or service principal' option is selected and click on 'Select members', search and select the SP 'AzureWeatherApp-SP-EUS' we created in the above steps.
                                                                                                                            
                                                                                                                                    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725197367593/5bfd47e6-5df2-4318-a368-8f79c4ee4b3a.png align="center")
                                                                                                                                            
                                                                                                                                                * Click on 'Review + assign'.
                                                                                                                                                        
                                                                                                                                                                ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725197566466/7105e168-7f9e-47be-9c20-d16083e1408f.png align="left")
                                                                                                                                                                        

                                                                                                                                                                        *Phew, that was a lot! But don't worry we are almost done*

                                                                                                                                                                        vi. Remember the client secret we created for our SP in step v? Let's make use of that now to log in to our ACR with Docker. You'll also need client ID which can be copied from the overview page of the 'AzureWeatherApp-SP-EUS' app registration.

                                                                                                                                                                        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725198750382/bb495b46-12c7-44dc-8059-8a7bdf9f985e.png align="center")

                                                                                                                                                                        vii. Execute the following command to login to ACR using Docker.

                                                                                                                                                                        *Note: Replace* ***&lt;sp-secret-value&gt;*** *and* ***&lt;sp-client-id&gt;*** *with the actual values of our service principal client id and client secret values.*

                                                                                                                                                                        ```bash
                                                                                                                                                                        docker login azureweatherappregistry.azurecr.io --username <sp-client-id> --password <sp-secret-value>
                                                                                                                                                                        ```

                                                                                                                                                                        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725199692069/dc9d3060-8403-4406-b46b-a7052ca6c97a.png align="center")

                                                                                                                                                                        viii. Tag the Docker image created in Step 1 with the fully qualified path to your ACR.

                                                                                                                                                                        ```bash
                                                                                                                                                                        docker tag azure-weather-app azureweatherappregistry.azurecr.io/azure-weather-app:latest
                                                                                                                                                                        ```

                                                                                                                                                                        Here, azureweatherappregistry.azurecr.io is the FQDN provided by Azure which follows the following convention &lt;your-acr-name&gt;.azurecr.io  
                                                                                                                                                                        We follow it by a slash and appending our previously created docker image name with the assigned tag &lt;your-acr-name&gt;.azurecr.io/&lt;your-docker-image-name&gt;:&lt;tag&gt;

                                                                                                                                                                        We can also get our ACR login server domain using the command below:

                                                                                                                                                                        ```bash
                                                                                                                                                                        az acr show --name azureweatherappregistry --query loginServer
                                                                                                                                                                        ```

                                                                                                                                                                        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725180059988/b768dc44-20d4-42c7-a1e9-874ac578059d.png align="center")

                                                                                                                                                                        ix. Next, use the Docker command below to push the Docker image we created during Step 1.

                                                                                                                                                                        ```bash
                                                                                                                                                                        docker push azureweatherappregistry.azurecr.io/azure-weather-app:latest
                                                                                                                                                                        ```

                                                                                                                                                                        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725200399906/9184cd9e-bc9f-4ab2-a1ba-67caf54691cd.png align="center")

### Step 3: Deploying application to Azure Container Instances

i. Run a command like the one below to start a container instance. Choose a `--dns-name-label` value that is unique within the Azure region where you're creating the instance. If you encounter a "DNS name label not available" error, try using a different DNS name label.

We opted for a public IP here to allow external access to the application, but you could also configure a private IP for internal usage scenarios.  
The image is set to the one we previously pushed to our ACR, and the port is configured to 80, where the Node.js application is running.

```bash
az container create --resource-group rg-az-weather-app --name aci-weather-app --image azureweatherappregistry.azurecr.io/azure-weather-app:latest --cpu 1 --memory 1 --registry-login-server azureweatherappregistry.azurecr.io --registry-username <sp-client-id> --registry-password <sp-secret-value> --ip-address Public --dns-name-label azurewebappcontainer --ports 80
```

*Note: Replace* ***&lt;sp-secret-value&gt;*** *and* ***&lt;sp-client-id&gt;*** *with the actual values of our service principal client id and client secret values.*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725206899955/4ced1302-4e33-444b-a664-ec63bd3bfb83.png align="center")

ii. To get the FQDN or the URL of our deployed container instance. Execute the following command.

```bash
az container show --resource-group rg-az-weather-app --name aci-weather-app --query ipAddress.fqdn
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725207340238/b0009231-5954-4b0a-99aa-224a18653794.png align="center")

iii. Follow the URL in your browser, you'll see a screen like below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725207405537/6c8e710c-c5ff-4da3-a8c9-71642937deab.png align="center")

iv. Since this is a weather app, enter the name of the city for which you'd like to view the weather forecast. For example, I'm using Kathmandu, the capital city of Nepal.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725207555388/85364bc5-5061-4f8f-84e4-e9785b8e4fe5.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725207568973/95408cac-5157-4b7e-ad9d-1438f69ea2e4.png align="center")

v. *Voilà!* we have successfully deployed the application and it's running on the cloud believe it or not.

In this tutorial, we learned how to containerize and deploy a Node.js application to Azure using Azure Container Instances and Azure Container Registry. By leveraging the power of Azure services, we simplified the entire deployment process, focusing on code and functionality rather than managing infrastructure. You can replicate this process with any Dockerized application to quickly deploy and test in the cloud.

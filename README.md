---
page_type: sample
description: "A minimal sample app that can be used to demonstrate deploying FastAPI apps to Azure App Service."
languages:
- python
products:
- azure
- azure-app-service
---

# Deploy a Python (FastAPI) web app to Azure App Service - Sample Application

This is the sample FastAPI application for the Azure Quickstart [Deploy a Python (Django, Flask or FastAPI) web app to Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/quickstart-python). For instructions on how to create the Azure resources and deploy the application to Azure, refer to the Quickstart article.

Sample applications are available for the other frameworks here:
- Django [https://github.com/Azure-Samples/msdocs-python-django-webapp-quickstart](https://github.com/Azure-Samples/msdocs-python-django-webapp-quickstart)
- Flask [https://github.com/Azure-Samples/msdocs-python-flask-webapp-quickstart](https://github.com/Azure-Samples/msdocs-python-flask-webapp-quickstart)

If you need an Azure account, you can [create one for free](https://azure.microsoft.com/en-us/free/).

## Local Testing

To try the application on your local machine:

### Install the requirements

`pip install -r requirements.txt`

### Start the application

`uvicorn main:app --reload`

### Example call

http://127.0.0.1:8000/

## Next Steps

To deploy your FastAPI app to Azure App Service using Azure Pipelines, you'll need to go through several steps, starting from creating the Azure App Service, configuring Azure DevOps to interact with Azure, pushing your code to Azure Repos, and finally setting up a pipeline for deployment.

### Full Deployment Process:

---

### **1. Create an Azure App Service**

1. **Login to Azure Portal**: 
   - Go to [Azure Portal](https://portal.azure.com).

2. **Create an App Service**:
   - In the portal, search for **App Services** and click **Create**.
   - Select **Subscription** and **Resource Group**.
   - Choose a **Region**.
   - For **App name**, choose a unique name for your FastAPI app (e.g., `fastapi-app-service`).
   - For **Publish**, select **Code**.
   - For **Runtime stack**, select **Python** (version 3.x).
   - Choose **Linux** as the operating system.
   - Click **Next**, review, and click **Create**.

---

### **2. Create a Service Connection in Azure DevOps**

1. **Go to Azure DevOps**:
   - Navigate to [Azure DevOps](https://dev.azure.com/) and select your **project**.

2. **Create a Service Connection**:
   - Click on the **Project settings** (gear icon) in the lower-left corner.
   - Under **Pipelines**, click **Service connections**.
   - Click **+ New service connection**.
   - Choose **Azure Resource Manager** and select **Service principal (automatic)**.
   - Sign in with your Azure account and grant Azure DevOps access to your Azure subscription.
   - Select the Azure subscription and **resource group** where your App Service is deployed.
   - Provide a **name** for the connection (e.g., `edureka-rg-dev-SCN`).
   - Click **Save**.

---

### **3. Push Your Code to Azure Repos**

1. **Go to Azure Repos**:
   - In Azure DevOps, navigate to **Repos** and create a new repository if you don't have one already.
   
2. **Initialize Git Locally**:
   - If you haven't already, initialize Git in your local project directory and connect it to Azure Repos.
   
   ```bash
   git init
   git remote add origin <your-azure-repo-url>
   git add .
   git commit -m "Initial commit"
   git push -u origin main
   ```

3. **Verify Local Running**:
   - Before pushing to Azure, ensure your FastAPI app works locally.
   
   Run your app with `uvicorn`:
   
   ```bash
   uvicorn main:app --reload
------------

### **4. Create a Pipeline in Azure DevOps**

1. **Create a New Pipeline**:
   - In Azure DevOps, go to **Pipelines** and click **New Pipeline**.
   - Choose **Azure Repos Git** and select your repository.
   - Click on **Starter Pipeline** or **YAML Pipeline**.
   
2. **Define the Pipeline Steps**:
   - Replace the default YAML with the following code, which installs dependencies, packages your FastAPI app, and deploys it to Azure.

### YAML Pipeline Example:

trigger:
  branches:
    include:
      - main  # This will trigger the pipeline on changes to the main branch

pool:
  vmImage: ubuntu-latest  # Using the latest Ubuntu VM image

variables:
  azureSubscription: 'edureka-rg-dev-SCN'  # The name of the Azure Service Connection
  appName: 'rmbchatbotapi'  # Name of your Azure App Service
  resourceGroup: 'edureka-rg-dev'  # Azure Resource Group
  packagePath: '$(System.DefaultWorkingDirectory)/app.zip'  # Path where the application zip will be stored

steps:
# Step 1: Set up Python environment
- task: UsePythonVersion@0
  inputs:
    versionSpec: '3.x'  # Specifies the Python version to be used (3.x)
    addToPath: true  # Adds Python to the PATH

# Step 2: Install dependencies
- script: |
    echo "Setting up Python environment..."
    python -m venv venv  # Creates a virtual environment
    source venv/bin/activate  # Activates the virtual environment
    pip install -r requirements.txt  # Installs dependencies listed in requirements.txt
  displayName: 'Install Dependencies'

# Step 3: Package the application into a zip file
- script: |
    echo "Packaging Application..."
    zip -r app.zip .  # Zips the entire application directory
  displayName: 'Package Application'

# Step 4: Publish pipeline artifact (optional, useful for debugging and storing artifacts)
- task: PublishPipelineArtifact@1
  inputs:
    targetPath: '$(Pipeline.Workspace)'  # Specifies the workspace where the artifact will be stored
    publishLocation: 'pipeline'  # Artifact will be stored as a pipeline artifact

# Step 5: Deploy the application to Azure App Service
- task: AzureWebApp@1
  inputs:
    azureSubscription: '$(azureSubscription)'  # The Azure service connection
    appType: 'webAppLinux'  # Specify webAppLinux for a Linux-based app
    appName: '$(appName)'  # The name of your Azure Web App
    package: '$(packagePath)'  # Path to the packaged application (zip file)
  displayName: 'Deploy to Azure App Service'

![image](https://github.com/user-attachments/assets/2ff43b85-75d6-43ee-a0d3-04093ce83229)

```
---

### **5. App Service Configuration**

1. **Startup Command**:
   - After deployment, you'll need to configure the **Startup Command** to run your FastAPI app correctly.
   - Go to your **App Service** in the Azure Portal.
   - Under **Settings**, select **Configuration**.
   - Add the following setting under **Application settings**:
   
     - **Name**: `STARTUP_COMMAND`
     - **Value**: `gunicorn -w 2 -k uvicorn.workers.UvicornWorker -b 0.0.0.0:8000 main:app`
   
   This tells Azure to use **Gunicorn** with **Uvicorn workers** to run your FastAPI app.
![image](https://github.com/user-attachments/assets/618c8739-8f58-4e69-bc88-3fcc4686512d)



3. **Add Additional Settings**:
   - In the **Application settings** section, add the following settings:
   
     - **Name**: `SCM_DO_BUILD_DURING_DEPLOYMENT`
     - **Value**: `1`
     - **Slot Setting**: `false`

     - **Name**: `WEBSITE_HTTPLOGGING_RETENTION_DAYS`
     - **Value**: `30`
     - **Slot Setting**: `false`
![image](https://github.com/user-attachments/assets/120be09a-afce-443c-b599-fd82df34717c)


---

### **6. Final Checks and Testing**

1. **Run the Pipeline**:
   - After setting up the pipeline, push code to the `main` branch in your Azure Repo. This will trigger the pipeline.
   
2. **Monitor the Pipeline**:
   - Monitor the progress of the pipeline in the **Pipelines** section of Azure DevOps. The pipeline will install dependencies, package the FastAPI app, set app settings, and deploy it to Azure.

3. **Verify Deployment**:
   - After deployment, navigate to the URL of your Azure App Service and verify that your FastAPI app is running.
   
   You should be able to see your FastAPI app responding to HTTP requests.

![image](https://github.com/user-attachments/assets/99eaed29-69f7-4b8d-8b51-cd2295c67b24)


---

### **Summary**

The complete process involves the following key steps:

1. **Create an Azure App Service** in the Azure Portal.
2. **Create a Service Connection** in Azure DevOps to link with Azure resources.
3. **Push your FastAPI code to Azure Repos** after ensuring it's working locally.
4. **Create a Pipeline** in Azure DevOps to automate deployment:
   - Install dependencies.
   - Package the app into a zip file.
   - Set necessary app settings (e.g., build during deployment, logging retention).
   - Deploy to Azure App Service.
5. **Configure App Service Settings** such as the startup command and environment variables.

This process will automate the deployment of your FastAPI app to Azure App Service using Azure DevOps.

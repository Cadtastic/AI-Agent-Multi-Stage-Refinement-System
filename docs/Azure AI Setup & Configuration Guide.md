# Azure AI Setup & Configuration Guide

## **1. Overview**

This guide outlines the steps to provision and configure **Azure AI Foundry**, a cloud-based platform for building and managing AI solutions, and **Azure OpenAI Cognitive Services**, which provides access to advanced language models like GPT-4 for deploying the AI Agent Multi-Stage Refinement System. This setup ensures a scalable, secure, and optimized environment for running multi-agent AI workflows, incorporating load balancing, encryption, and compliance with security best practices, such as GDPR and HIPAA where applicable, through secure data handling policies and end-to-end encryption.

## **2. Prerequisites**

Before proceeding, ensure you have:

- An **Azure Subscription** with administrative privileges (e.g., Pay-As-You-Go, Enterprise Agreement).
- Access to **Azure AI Foundry** and **Azure OpenAI Cognitive Services**.
- Installed **Azure CLI** and **PowerShell (Optional)**.
- A **resource group** for organizing Azure resources.

---

## **3. Creating Azure AI Foundry Resources**

### **Step 1: Sign in to Azure**

```sh
az login
```

This command logs you into your Azure account. If you have multiple subscriptions, select the correct one:

```sh
az account set --subscription ""
```

### **Step 2: Create a Resource Group**

Create a new resource group to manage all AI-related services.

```sh
az group create --name AI-Resource-Group --location eastus
```

### **Step 3: Deploy Azure AI Foundry**

```sh
az cognitiveservices account create \
--name AI-Foundry-Instance \
--resource-group AI-Resource-Group \
--kind OpenAI \
--sku S0 \
--location eastus \
--yes
```

This provisions an **Azure AI Foundry** instance with OpenAI capabilities, enabling natural language processing, text generation, and AI-powered automation.

---

## **4. Deploying & Configuring Azure OpenAI Models**

### **Step 1: Create an Azure OpenAI Service**

```sh
az cognitiveservices account create \
--name OpenAI-Instance \
--resource-group AI-Resource-Group \
--kind OpenAI \
--sku S0 \
--location eastus \
--yes
```

### **Step 2: Deploy a Language Model**

Once the service is created, deploy a model such as **GPT-4**. Alternative models like GPT-3.5 or Codex may be preferable for specific use cases. GPT-3.5 is a cost-effective option for general-purpose tasks, such as chatbot interactions and summarization, while Codex excels at code generation, making it ideal for assisting with software development, debugging, and code completion.

```sh
az cognitiveservices account deployment create \
--name OpenAI-Instance \
--resource-group AI-Resource-Group \
--model-name gpt-4 \
--model-version "latest" \
--sku S0
```

### **Step 3: Retrieve API Keys & Endpoints**

To interact with the OpenAI service, retrieve the API key and endpoint. It is recommended to store API keys securely using Azure Key Vault or environment variables.

```sh
az cognitiveservices account keys list --name OpenAI-Instance --resource-group AI-Resource-Group
```

---

## **5. Setting Up API Authentication & Access Control**

### **Step 1: Enable Role-Based Access Control (RBAC)**

Assign role permissions for secure API access. Role-Based Access Control (RBAC) is necessary to manage permissions effectively, controlling actions such as model deployment, API key retrieval, and access to inference endpoints.

```sh
az role assignment create --assignee \
--role "Cognitive Services OpenAI User" \
--scope /subscriptions//resourceGroups/AI-Resource-Group
```

### **Step 2: Secure API Access with Managed Identity**

Use **Managed Identity** to authenticate AI services without exposing API keys.

```sh
az identity create --name OpenAIIdentity --resource-group AI-Resource-Group
```

Retrieve the identity's principal ID using the following command:
```sh
az identity show --name OpenAIIdentity --resource-group AI-Resource-Group --query 'principalId' --output tsv
```
Example output:
```sh
12345678-1234-1234-1234-123456789abc
```
Then, assign access:
```sh
az identity show --name OpenAIIdentity --resource-group AI-Resource-Group --query 'principalId' --output tsv
```
Then, assign access:

```sh
az role assignment create --assignee "" \
--role "Cognitive Services OpenAI User" \
--scope /subscriptions//resourceGroups/AI-Resource-Group
```

---

## **6. Testing API Endpoints for Model Invocation**

### **Step 1: Install OpenAI Python SDK**

```sh
pip install openai
```

### **Step 2: Run a Test API Call**

Create a Python script to verify AI model deployment. Ensure API keys are securely stored in environment variables instead of hardcoding them.

```python
import openai

openai.api_type = "azure"
openai.api_base = "https://.openai.azure.com/"
openai.api_version = "2023-05-15"
openai.api_key = ""

response = openai.ChatCompletion.create(
engine="gpt-4",
messages=[
{"role": "system", "content": "You are an AI assistant."},
{"role": "user", "content": "Hello, how are you?"}
]
)

print(response["choices"][0]["message"]["content"])
```

Run the script to confirm successful AI service setup. If any errors occur, check for incorrect API keys, misconfigured endpoints, or insufficient role permissions in Azure. Consider adding logging and exception handling in the script to improve debugging efficiency. A successful response should return a well-formatted AI-generated message based on the input prompt.

```sh
python test_openai.py
```

---

# Cloud-Native App for Best Buy
Welcome to the Best Buy application.

This sample demo app consists of a group of containerized microservices that can be easily deployed into a Kubernetes cluster. This is meant to show a realistic scenario using a polyglot architecture, event-driven design, and common open source back-end services (eg - Azure Service Bus, MongoDB). The application also leverages OpenAI's models to generate product descriptions and images. This can be done using either [Azure OpenAI](https://learn.microsoft.com/azure/ai-services/openai/overview) or [OpenAI](https://openai.com/).

This application is inspired by Azure Kubernetes Service (AKS) 
 

The **Best Buy App** must include the following components:  

| Service              | Description                                                                 | Notes                                      |
|----------------------|-----------------------------------------------------------------------------|--------------------------------------------|
| **Store-Front**      | Customer-facing app for browsing and placing orders.              |                                           |
| **Store-Admin**      | Employee-facing app for managing products and viewing orders .     |                                           |
| **Order-Service**    | Handles order creation and sends data to the managed order queue.          | Replace RabbitMQ with a managed service.  |
| **Product-Service**  | Handles CRUD operations for product data.                          |                                           |
| **Makeline-Service** | Processes and completes orders by reading from the order queue.       |                                           |
| **AI-Service**       | Generates product descriptions and images.                       | Use GPT-4 and DALL-E models.              |
| **Database**         | MongoDB for persisting order and product data.                            |                                           |

---

   - **Updated Application Architecture**:  
      ![Algonquin Pet Store On Steroids](https://github.com/user-attachments/assets/6a2a8084-5b1e-48da-95d9-e3d2722ad02d)
   - **Application and Architecture Explanation**:  
   

# Application Functionality and Architecture Overview

## Functionality
1. **Customer Interface**:
   - Customers engage through the store-front, built with **Vue.js**, to place orders and browse products.
2. **Employee Interface**:
   - Employees use the store-admin, also developed with **Vue.js**, to manage orders and oversee operations.
3. **Order Processing**:
   - Orders flow through the system, leveraging efficient services for processing and database updates.
4. **AI Features**:
   - Integrates advanced AI capabilities such as language understanding and image generation.

## Architecture
1. **Core Services**:
   - **Order-Service**: Handles orders using **Node.js** and communicates through **Azure Service Bus**.
   - **Product-Service**: Manages product information via **Rust**.
   - **Makeline-Service**: Ensures smooth order processing and updates the **MongoDB** database (developed with **Go**).
   - **AI-Service**: Powers language and image functionalities using models like **GPT-4** and **DALL-E** (built in **Python**).

2. **Interaction**:
   - The store-front connects customers to relevant services.
   - The store-admin enables employees to work efficiently with system tools.

3. **Databases and Models**:
   - Utilizes **MongoDB** for order storage.
   - Leverages AI models for enhanced functionalities.

---

This architecture showcases a thoughtful design, leveraging diverse frameworks and programming languages to create a robust and scalable application.

   - **Deployment Instructions**:  
    # Apply the YAML File to the AKS Cluster


# Step 1: Create an Azure Kubernetes Cluster (AKS)

## Log in to Azure Portal
1. Go to [Azure Portal](https://portal.azure.com) and log in with your Azure account.

## Create a Resource Group
1. In the Azure Portal, search for **Resource Groups** in the search bar.
2. Click **Create** and fill in the following:
   - **Resource group name**: ` CST8915-FINAL-PROJECT`
   - **Region**: Canada
3. Click **Review + Create** and then **Create**.

## Create an AKS Cluster
1. In the search bar, type **Kubernetes services** and click on it.
2. Click **Create** and select **Kubernetes cluster**.
3. In the **Basics** tab, fill in the following details:
   - **Subscription**: Select your subscription.
   - **Resource group**: Choose `CST8915-FINAL-PROJECT`.
   - **Cluster preset configuration**: Choose **Dev/Test**.
   - **Kubernetes cluster name**: `BestBuyCluster`.
   - **Region**: Same as your resource group (e.g., Canada).
   - **Availability zones**: None.
   - **AKS pricing tier**: Free.
   - **Kubernetes version**: Default.
   - **Automatic upgrade**: Disabled.
   - **Automatic upgrade scheduler**: No schedule.
   - **Node security channel type**: None.
   - **Security channel scheduler**: No schedule.
   - **Authentication and Authorization**: Local accounts with Kubernetes RBAC.

4. In the **Node pools** tab, fill in the following details:
   - **Node pool name**: Select `agentpool`. Optionally, change its name to `masterpool`. These nodes will have the control plane.
   - **Node size**: `D2as_v4`.
   - **Scale method**: Manual.
   - **Node count**: 1.
5. Click **Update**.

6. Click **Add node pool**:
   - **Node pool name**: `workerspool`.
   - **Mode**: User.
   - **Node size**: `D2as_v4`.
   - **Scale method**: Manual.
   - **Node count**: 1.
5-Set the cluster subscription using the command shown in the portal :
6-Copy the command shown in the portal for configuring kubectl

after create Cluster and connected

1. Use the K8s deployment YAML files: in "Deployment Files"
   
2. Open the terminal and navigate to the file directory in VS code.
   
3. Run the following commands to apply the YAML configuration:
   
```
kubectl apply -f service-config.yaml
kubectl apply -f azure-servicebus-secret.yaml

kubectl apply -f mongodb.yaml
kubectl apply -f order-service-deployment.yaml
kubectl apply -f product-service-deployment.yaml
kubectl apply -f store-admin-service-deployment.yaml
kubectl apply -f store-front-service-deployment.yaml
kubectl apply -f makeline-service-deployment.yaml
kubectl apply -f ai-service.yaml
```


4-Verify the deployment:

```
kubectl get pods
```
5- Check services:

```
kubectl get services
```
# Step 2: Create ConfigMaps and Secrets for managing application configurations and sensitive information.
# Azure Service Bus Setup for Best Buy Project (Azure Portal)

This guide walks you through creating an Azure Service Bus namespace and queues using the Azure Portal. These queues will be used by your sender and receiver apps (running separately, such as in AKS).

---

## 1. Sign in to Azure Portal

Go to [https://portal.azure.com](https://portal.azure.com) and log in with your Azure account.

---

## 2. Create a Resource Group (if you don't have one)

1. In the top search bar, type **Resource Groups** and click the result.
2. Click **+ Create**.
3. Fill in the following:
   - **Subscription**: Choose your subscription
   - **Resource Group Name**: 
   - **Region**: `Canada Central` (or your preferred region)
4. Click **Review + Create**, then **Create**.

---

## 3. Create a Service Bus Namespace

1. In the top search bar, type **Service Bus** and click **Service Bus Namespaces**.
2. Click **+ Create**.
3. Under the **Basics** tab:
   - **Subscription**: Your subscription
   - **Resource Group**: 
   - **Namespace Name**: 
   - **Region**: `Canada Central`
   - **Pricing Tier**: Select **Standard** (to enable multiple queues)
4. Click **Review + Create**, then **Create**.

Deployment will take a few moments.

---

## 4. Create Queues

1. After the namespace is created, go to **BestBuyServiceBus** (click “Go to resource”).
2. In the left menu, select **Queues**.
3. Click **+ Queue** to add a new queue:
   - Name: `orders`
   - Leave default settings unless you need dead-lettering or TTL.
   - Click **Create**

---

## 5. Get the Connection String

1. In your namespace, go to **Shared access policies** (left menu).
2. Click on **RootManageSharedAccessKey**.
3. Copy the **Primary Connection String** (you'll use this in your sender and receiver apps).

> 🔐 Save this securely or store it as a Kubernetes Secret if used in AKS.

---

## 6. Monitor Queues

1. Go to the **Queues** section of your namespace.
2. Click on a queue (e.g., `orders) to view:
   - Active messages
   - Dead-letter messages
   - Message count graph
   - Delivery attempts

## 7. Task 4: Update the Deployment and Configuration files in the Deployment Files folder.

# step 3 Use StatefulSets for MongoDB.

   - **Table of Microservice Repositories**:  
     - A table listing each microservice repository and its GitHub link.  
     - Example:
       | **Service**         | **Repository Link**                       |
       |---------------------|-------------------------------------------|
       | Store-Front         | (https://github.com/MteacherIT/store-front-best-buy)                         |
       | Store-admin         | (https://github.com/MteacherIT/store-admin-best-buy)                         |
       | Order-Service       | (https://github.com/MteacherIT/order-service-best-buy)                          |
       | Product-Service      | (https://github.com/MteacherIT/product-service-best-buy)                           |
       | Makeline-Service        | (https://github.com/MteacherIT/makeline-service-best-buy)                          |
       | AI-Service       |(https://github.com/MteacherIT/ai-service-best-buy)                           |
       | Database       |(https://github.com/MteacherIT/mongo-best-buy)                          |
   - **Table of Docker Images**:  
     - A table listing all Docker images you created, including their names and links to their Docker Hub repositories.  
     - Example:
       | **Service**         | **Docker Image Link**                     |
       |---------------------|-------------------------------------------|
        | Store-Front         | (https://hub.docker.com/r/ma7aara/store-front-bb)                           |
       | Store-admin         | (https://hub.docker.com/r/ma7aara/store-admin-bb)                           |
       | Order-Service       | (https://hub.docker.com/r/ma7aara/order-service-bb)                          |
       | Product-Service      | (https://hub.docker.com/r/ma7aara/product-service-bb)                           |
       | Makeline-Service        | (https://hub.docker.com/r/ma7aara/makeline-service-bb)                           |
       | AI-Service       | (https://hub.docker.com/r/ma7aara/ai-service-bb)                           |
      
    
 

### **2. Demo Video**  

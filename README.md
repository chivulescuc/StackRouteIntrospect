# 🚀 Deploying Containerized .NET Microservices with Dapr on Azure Container Apps

This project demonstrates how to **containerize .NET microservices**, deploy them to **Azure Container Apps (ACA)**, and enable **event-driven communication** using **Dapr Pub/Sub**.
Additionally, Azure OpenAI can be used to review and troubleshoot deployments by suggesting improvements such as retry logic, telemetry points, and scaling strategies.

---

## 📌 Architecture

- **ProductService** → Publishes product events.
- **OrderService** → Subscribes to product events.
- **Azure Container Registry (ACR)** → Stores Docker images.
- **Azure Container Apps (ACA)** → Runs services with Dapr sidecars.
- **Dapr Pub/Sub** → Enables messaging between services (in-memory for demo).

```
+----------------+       Pub/Sub       +----------------+
| ProductService |  ---------------->  |  OrderService  |
|   (Publisher)  |                     | (Subscriber)   |
+----------------+                     +----------------+
         |                                       |
     Docker Image                           Docker Image
         |                                       |
        ACR  <-------- Deployed to --------->  ACA
```

---

## 📂 Project Structure

```
.
├── ProductService/
│   ├── Program.cs
│   ├── Dockerfile
│   └── ProductService.csproj
├── OrderService/
│   ├── Program.cs
│   ├── Dockerfile
│   └── OrderService.csproj
├── in-memory-pubsub.yaml   # Dapr pub/sub component
└── README.md               # This file
```

---

## ⚙️ Prerequisites

- Azure Subscription
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [.NET SDK](https://dotnet.microsoft.com/en-us/download)
- [Docker](https://docs.docker.com/get-docker/)

---

## 🚀 Deployment Steps

## 🚀 Steps to Deploy

### 1. Prerequisites
- Azure CLI installed
- Docker installed (local dev)
- Azure subscription with permissions
- Azure Container Apps extension for CLI:
  ```bash
  az extension add --name containerapp --upgrade

### 1. Create Resource Group
```bash
az group create -n introspect_b_grp -l eastus
```

### 2. Create ACR
```bash
ACR_NAME=introspectacr
az acr create -g introspect_b_grp -n $ACR_NAME --sku Basic
az acr update -n $ACR_NAME --admin-enabled true
```
### 3. Build & Push Docker Images

2. Containerize Services
Dockerfile (for each service)
```
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY *.csproj ./
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "ProductService.dll"]  # or OrderService.dll

bash
# ProductService
cd ProductService
docker build -t $ACR_NAME.azurecr.io/productservice:latest .
docker push $ACR_NAME.azurecr.io/productservice:latest

# OrderService
cd ../OrderService
docker build -t $ACR_NAME.azurecr.io/orderservice:latest .
docker push $ACR_NAME.azurecr.io/orderservice:latest
```

### 4. Create ACA Environment
```bash
az containerapp env create -g introspect_b_grp -n introspect-env -l eastus 2
```

### 5. Deploy Services with Dapr Enabled
```bash
# ProductService
az containerapp create -g introspect_b_grp -n productservice   --environment introspect-env   --image $ACR_NAME.azurecr.io/productservice:latest   --target-port 5000 --ingress internal   --dapr-enabled true --dapr-app-id productservice --dapr-app-port 5000

# OrderService
az containerapp create -g introspect_b_grp -n orderservice   --environment introspect-env   --image $ACR_NAME.azurecr.io/orderservice:latest   --target-port 5000 --ingress internal   --dapr-enabled true --dapr-app-id orderservice --dapr-app-port 5000
```

### 6. Configure Pub/Sub Component (In-Memory)
```bash
az containerapp env dapr-component set   -n introspect-env -g introspect_b_grp   --dapr-component-name pubsub   --yaml in-memory-pubsub.yaml
```

---

## 📊 Monitoring & Logs

```bash
az containerapp logs show -n productservice -g introspect_b_grp --follow
az containerapp logs show -n orderservice -g introspect_b_grp --follow
```

---

## 🎯 Learning Outcomes

- Understand **ACA deployment**
- Implement **service-to-service messaging with Dapr**
- Observe **distributed service communication and logs**


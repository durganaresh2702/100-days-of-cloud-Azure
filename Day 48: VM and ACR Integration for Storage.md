# Day 48: VM and ACR Integration for Storage

The Nautilus DevOps team needs to set up an Azure Virtual Machine (VM) to interact with an Azure Blob Storage container for storing and retrieving data. The team must create a private storage account, configure Blob Storage, and test the functionality.
Task:

1. **Azure Virtual Machine Setup:**

   - Create a VM named `xfusion-vm` in the `East US` region.
   - Authentication: Use SSH public key authentication. (Please select use existing public key option, create `public-key` locally and paste contents of `~/.ssh/id_rsa.pub`).
   - Install `Docker` and `Azure CLI` on the VM.
   - Pull the Docker image from the ACR and run it on the VM, ensuring it listens on port `80`.

2. **Azure Container Registry (ACR) Setup:**

   - Create an ACR named `xfusionacr20684` in the `East US` region.
   - The repository name should be `xfusion/python-app`.
   - Build the Docker image using the `Dockerfile` already given under `/root/pyapp`.
   - Push the Docker image to the ACR with the tag `latest`.

3. **Create a Storage Account and Blob Container:**

   - Create a storage account named `xfusionstor20684` in the `East US` region with `Locally-redundant storage (LRS)`.
   - Create a Blob container named `xfusion-config`.
   - Upload a file named `config.json` (available under `/root/`) to the container.

4. **Validation:**

   - Confirm that the application can be accessed on the browser.



## Task 1: Create an Azure Virtual Machine

1. **From Azure portal, select virtual machines.**

   <img width="1010" height="143" alt="image" src="https://github.com/user-attachments/assets/bd04d0a5-908b-4e24-b93d-4ad051db20c6" />

2. **In the Virtual machines page, select Create and then Azure virtual machine. The Create a virtual machine page opens.**

   <img width="1085" height="411" alt="image" src="https://github.com/user-attachments/assets/e69caf0e-fe3a-4b4b-9202-44c588edb883" />

3. **Click `+ Create` and select `Virtual machine`.**

   <img width="317" height="412" alt="image" src="https://github.com/user-attachments/assets/35951587-5317-484a-8ebc-53acb8a0057f" />

4. E**nter VM name and choose region in `East US` and select `Size` as `Standard_B1s`.**

   <img width="1348" height="415" alt="image" src="https://github.com/user-attachments/assets/5f747ef4-9a3f-4501-aeff-ffbd742ebcc9" />

5. **For authentication type, select SSH and `Existing SSH key`.**

   <img width="1346" height="378" alt="image" src="https://github.com/user-attachments/assets/cb869bbc-cc18-406b-ab91-b8f72ec6e531" />

6. **Allow both SSH and HTTP for inbound port.**

   <img width="1346" height="252" alt="image" src="https://github.com/user-attachments/assets/be05f7c0-082d-45c0-a700-f5774184c2d2" />

7. On the `Disk` tab, select `OS disk type` as `Standard_HDD`.

   <img width="1346" height="291" alt="image" src="https://github.com/user-attachments/assets/fe6a8fcd-c521-4734-9ed5-335c2b0d08f0" />

8. **Review and Create.**

   <img width="1098" height="325" alt="image" src="https://github.com/user-attachments/assets/e6472973-8e71-477a-9133-f9f063805326" />


## Task 2: Azure Container Registry (ACR) Setup

1. **From all resources, select `Container registries`.**

   <img width="1107" height="351" alt="image" src="https://github.com/user-attachments/assets/595aae74-a651-434f-8d87-2c4fbbece6b5" />

2. **CLick on `+ Create`.**

   <img width="1365" height="497" alt="image" src="https://github.com/user-attachments/assets/91ad28dc-b245-45ee-9fe5-5488e1b28296" />

3. **Enter registry name, choose region in `East US` and `Pricing plan` as `Basic`.**

   <img width="1348" height="354" alt="image" src="https://github.com/user-attachments/assets/880eeef6-9fe6-49ab-a859-1f9d7c2c753c" />

4. **On the `Review + create` page, click `Create`.**

   <img width="1348" height="452" alt="image" src="https://github.com/user-attachments/assets/6da7fe4c-e487-4a33-a092-a65bc6fcb4c7" />


## Task 3: Build & Push Docker Image to Registry on Azure Client Host

1. **Login to Azure Container Registry:**

   ```sh
   az acr login --name xfusionacr20684
   ```

2. **Go to app directory:**

   ```sh
   cd /root/pyapp
   ```

3. **Build docker image from Dockerfile:**

   ```sh
   docker build -t xfusionacr20684.azurecr.io/xfusion/python-app:latest .
   ```

4. **Push image to container regisry:**

   ```sh
   docker push xfusionacr20684.azurecr.io/xfusion/python-app:latest
   ```

5. **Verify image uploaded in the registry from CLI:**

   ```sh
   az acr repository list --name xfusionacr20684 --output table
   ```

6. **Or verify from Azure Portal:**

   <img width="846" height="350" alt="image" src="https://github.com/user-attachments/assets/82aa861e-ec32-43b9-b054-853651025f83" />


## Task 4: Steup Virtual machine for Docker and Azure CLI

1. **SSH into the Virtual machine.**

   ```sh
   ssh azureuser@<public-ip>
   ```

3. **Install Docker Virtual machine.**

   ```sh
   sudo apt update
   sudo apt install -y docker.io
   sudo systemctl enable docker
   sudo systemctl start docker
   sudo usermod -aG docker azureuser
   exit
   ```

>[!Note]
> **Connect to Virtual machine again for new session.**

3. **Verify Docker installation on Virtual machine:**

   ```sh
   docker --version
   ```

4. **Install Azure CLI on Virtual machine.**

   ```sh
   curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
   ```
   
5. **Verify Azure CLI installed on Virtual machine:**

    ```sh
    az version
    ```

6. **Login to Azure:**

   ```sh
   az login
   ```


## Task 5: Run Docker Container on Virtual machine.

1. **Retrive registry credential:**

   <img width="217" height="485" alt="image" src="https://github.com/user-attachments/assets/102d4668-e7cb-4868-94df-b27bc7d39edb" />

2. **ACR → Access keys → Enable Admin user:**

   <img width="846" height="270" alt="image" src="https://github.com/user-attachments/assets/04eeadb5-2d10-4abc-96d5-7a06f50cd559" />


3. **Login to registry using CLI for container pulling:**

   ```sh
   docker login xfusionacr20684.azurecr.io
   ```

   ```sh
   Username: xfusionacr20684
   Password: <admin-password>
   ```

4. **Copy `config.json` to VM from azure client host.**
   
5. **Run Container on Port `80` on Virtual machine.**

   ```sh
   docker run -d \
     -p 80:80 \
     -v /home/azureuser/config.json:/app/config.json \
     --name xfusion-app \
     xfusionacr20684.azurecr.io/xfusion/python-app:latest
   ```


6. **Ensure container is running:**

   ```sh
   docker ps
   ```

   ```sh
   CONTAINER ID   IMAGE                                                  COMMAND           CREATED         STATUS         PORTS                                 NAMES
   557889cd55f3   xfusionacr20684.azurecr.io/xfusion/python-app:latest   "python app.py"   8 seconds ago   Up 7 seconds   0.0.0.0:80->80/tcp, [::]:80->80/tcp   xfusion-app
   ```

## Task 6: Storage Account & Blob Container

1. **Create storage account.**

   <img width="1350" height="410" alt="image" src="https://github.com/user-attachments/assets/e3726f0d-0c41-41e1-8d0b-8028bf143983" />

2. `Select `Blob container` under storage account.`

   <img width="1089" height="406" alt="image" src="https://github.com/user-attachments/assets/0ce852c6-91ce-4a7f-9bff-e509470908c6" />

3. **Click on `Create container`.**

   <img width="862" height="210" alt="image" src="https://github.com/user-attachments/assets/89c13314-d39c-441d-82be-3ae3f042727c" />

4. **Enter container name.**

   <img width="389" height="579" alt="image" src="https://github.com/user-attachments/assets/18ece14e-849c-4718-a949-61af360100dd" />

5. **Preview after creating container.**

   <img width="1128" height="303" alt="image" src="https://github.com/user-attachments/assets/00953f78-ff4a-4c37-8150-35499c37dd3b" />

6. **Upload `config.json` to container using Azure portal.**

   <img width="1128" height="325" alt="image" src="https://github.com/user-attachments/assets/4bb8949d-e850-4725-b9c7-8abbab5a3f52" />


## Task 6: Verify web access

1. Enter VM public IP Address in a browser.

   <img width="936" height="131" alt="image" src="https://github.com/user-attachments/assets/669af4c4-3962-4386-af46-8721a3200547" />

az acr login --name nautilusacr25952
 
cd /root/pyapp

docker build -t nautilusacr25952.azurecr.io/nautilus/python-app:latest .

docker push nautilusacr25952.azurecr.io/nautilus/python-app:latest

az acr repository list --name nautilusacr25952 --output table

ssh azureuser@20.127.104.161

sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker azureuser
exit

curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

az version

az login

docker login nautilusacr25952.azurecr.io

username:nautilusacr25952
password:BNiOVm98z1P5XkRx8DUKlRgbmjSYaQMW2i0TbznNYuC03gFXBURhJQQJ99CFACYeBjFEqg7NAAACAZCRTvq1


sudo scp -i /root/.ssh/id_rsa -o StrictHostKeyChecking=no /root/config.json azureuser@20.127.104.161:/home/azureuser/config.json

docker run -d \
  -p 80:80 \
  -v /home/azureuser/config.json:/app/config.json \
  --name nautilus-app \
  nautilusacr25952.azurecr.io/nautilus/python-app:latest

docker ps

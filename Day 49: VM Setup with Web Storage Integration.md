# Day 49: VM Setup with Web Storage Integration

The Nautilus DevOps team is tasked with setting up an environment to host a static web application. The application will serve static content from an Azure Storage Account, and a Virtual Machine (VM) will be configured to fetch and display this content using `Nginx`. The Azure Storage Account is used as a secure, centralized location for storing the index.html file. The team intentionally keeps this file outside the main source code repository, since that repository contains additional internal application code that should not be exposed to or accessed by the VM. By placing only the required static file in the Storage Account, the team can distribute this asset safely and independently of the full codebase.

The VM should securely download the `index.html` blob directly from the designated container (e.g., using Azure CLI, SAS URL, or REST API) and place it in Nginx’s web root directory so that it is served locally by `Nginx`. The Storage Account is not mounted, and the Static Website feature is not used. The VM retrieves the file during deployment and may re-fetch it whenever updates are needed. The resources must follow best practices for security, performance, and accessibility.

**Task Details:**

1) **Create a Virtual Network (VNet) and Subnet:**

   - Create a VNet named `xfusion-vnet` in the `East US` region.
   - Create a subnet named `xfusion-subnet` within the VNet for the VM.

2) **Create an Azure Storage Account:**

    - Create a storage account named `xfusionstor31102` in the `East US` region with `Locally-redundant storage (LRS)`.
    - Create a Blob container named `xfusion-container` in the storage account.
    - Upload the `index.html` file located at `/root` on the client host to the container `xfusion-container`.
    - Ensure the Storage Account is private and not publicly accessible by disabling public access for the storage account.

3) **Create a Virtual Machine (VM):**

    - Create a VM named `xfusion-vm` in the `East US` region.
    - Use the `xfusion-vnet` and subnet `xfusion-subnet` for the VM.
    - Authentication: Use SSH public key authentication. (Please select use existing public key option, create public-key locally and paste contents of `~/.ssh/id_rsa.pub`)

    - Install `Nginx` on the VM.

    - Ensure `Nginx` is configured to serve the file from `/var/www/html/index.html`.

4) **Verify Setup:**

    - Verify that the Nginx web server on the client host serves the index.html file correctly when accessing the VM's public IP address.


# Task 1: Create a Virtual Network (VNet) and Subnet

1. **Go to Azure Portal → Virtual networks → Create.**

   <img width="1327" height="366" alt="image" src="https://github.com/user-attachments/assets/66747051-85c6-434c-8e87-5a1118332be9" />

2. **Click `Next`: IP Addresses Click `Add subnet`.**
   
   - Subnet name: `xfusion-subnet`
   - Subnet range: `10.0.1.0/24`

   <img width="523" height="579" alt="image" src="https://github.com/user-attachments/assets/a433c798-921d-4175-b13c-f6a370195b3f" />

3. **Review + Create → Create**

   <img width="653" height="254" alt="image" src="https://github.com/user-attachments/assets/65cfdfaf-c027-4f1c-b7c4-d1759e0ec237" />

## Task 2: Create Storage Account & Upload file

1. Storage accounts → Create. On the `Basics` tab.

   - Name: `xfusionstor31102`
   - Region: `East US`
   - Performance: `Standard`
   - Redundancy: `LRS`
     
    <img width="1348" height="402" alt="image" src="https://github.com/user-attachments/assets/61bf9670-bf3a-4b5f-89d4-95359279d234" />

3. **Create Blob Container Open `xfusionstor31102` Go to Containers Click + Container.**

   - Name: `xfusion-container`
   - Public access level: `Private (no anonymous access)`

   <img width="389" height="579" alt="image" src="https://github.com/user-attachments/assets/73da77db-5680-4161-ac1d-70aa036b9a3d" />

4. **Create and review container after creation.**

   <img width="1128" height="317" alt="image" src="https://github.com/user-attachments/assets/334a5cd0-9e10-487c-85ae-c9dabfc98c0a" />

5. **Upload index.html file to the container using azure portal.**

   <img width="1128" height="317" alt="image" src="https://github.com/user-attachments/assets/dd409fbf-6e5d-49b3-8dea-af68239f2a25" />

## Task 3: Create a Virtual Machine (VM

1. **Generate ssh keys in azure client host.**

   ```sh
   ssh-keygen -t rsa
   ```

2. **Create virtual machine in `East US` region where `Size` will be `Standard_B1s`.**

   <img width="1346" height="406" alt="image" src="https://github.com/user-attachments/assets/8db3b314-5ef6-4d4a-a214-c5df1d484d9b" />

3. **Authentication: SSH public key SSH public key: Paste `id_rsa.pub`.**

   <img width="1348" height="380" alt="image" src="https://github.com/user-attachments/assets/9dcba4b9-5d31-464b-936b-e076c225f5cb" />

4. **Inbound port rule should have both `SSH `and `HTTP`.**

   <img width="1346" height="255" alt="image" src="https://github.com/user-attachments/assets/7fd131cf-946f-47fb-946d-340f87bc832e" />

5. On the `Disk` tab, choose `OS disk type` as `Standard HDD`.

   <img width="1346" height="185" alt="image" src="https://github.com/user-attachments/assets/216d8ec5-44e4-4ae6-8404-e766b8785141" />

6. **On the `Networking` tab, select following:**
   
   - Virtual network: `xfusion-vnet`
   - Subnet: `xfusion-subnet`
   - Public IP: `Enabled`

   <img width="1346" height="412" alt="image" src="https://github.com/user-attachments/assets/26c96175-1217-4fb6-93d4-b40ebd113e9e" />

7. **Review + Create → Create.**

   <img width="1096" height="264" alt="image" src="https://github.com/user-attachments/assets/e993649b-bb8a-4f1b-bf72-b1536a3fe893" />


## Task 3: Virtual machine setup for website

1. **SSH into virtual machine from azure client host.**

   ```sh
   sssh azureuser@<pub-ip>
   ```
   
3. **Install, start and enable Nginx on the xfusion virtual machine.**

   ```
   sudo apt update -y
   sudo apt install -y nginx
   sudo systemctl start nginx
   sudo systemctl enable nginx
   ```

4. **Verify Nginx installed and started in CLI.**

   <img width="1262" height="275" alt="image" src="https://github.com/user-attachments/assets/106d6a3f-55f6-45b1-ade0-b8769a158dde" />

5. **Install Azure CLI on Virtual machine.**

   ```sh
   curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
   ```

6. **Login to Azure in Virtual machine.**

   ```sh
   az login
   ```

8. **Get Storage Account Key from Azure Portal.**

   <img width="1342" height="441" alt="image" src="https://github.com/user-attachments/assets/723a60eb-df74-4b33-9142-90928300f643" />

7. **Download Blob to Nginx Web Root in Virtual machine.**

   ```sh
   sudo az storage blob download \
     --account-name xfusionstor31102 \
     --account-key <STORAGE_ACCOUNT_KEY> \
     --container-name xfusion-container \
     --name index.html \
     --file /var/www/html/index.html
   ```

8. **Verify Nginx Configuration.**

   ```sh
   sudo nginx -t
   sudo systemctl reload nginx
   ```

9. **Open browser and enter public ip of the Virtual machine.**

   <img width="1117" height="176" alt="image" src="https://github.com/user-attachments/assets/14b0ace0-f18f-4ac8-bf9a-68fc2afcddd7" />


 **Correct commands**

ssh azureuser@20.124.103.191

sudo apt update -y
sudo apt install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx

curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

az login

get access key from storage browser

sudo az storage blob download --account-name nautilusstor6851 --account-key <straoge_account_key> --container-name nautilus-container --name index.html --file /var/www/html/index.html

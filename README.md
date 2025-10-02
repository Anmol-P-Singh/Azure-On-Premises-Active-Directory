# 🏗️ Deploying Active Directory in Azure: A Step-by-Step Guide Using Virtual Machines

<p align="center">
  <img width="300" height="168" alt="Active Directory" src="https://github.com/user-attachments/assets/ab43591a-2189-41f9-875f-b6e6c8971685" />
</p>

This tutorial will help you guide through the initial setup of Active Directory on Azure Virtual Machines

---

## What You’ll Be Using

- Microsoft Azure (Virtual Machines and Networking)
- Remote Desktop Protocol (RDP)
- Active Directory Domain Services (AD DS)
- PowerShell

---

## Operating Systems We’ll Work With

- Windows Server 2022 (for the Domain Controller)
- Windows 10 (21H2) (for the Client)

---

## Step 1: Create a Resource Group

- Head over to the **Azure Portal**.
- Navigate to **Resource Groups** and hit *Create*.
- Give it a name — I went with `Active-D`.
- Pick your favorite region (like `East US`).

![Create Resource Group](https://github.com/user-attachments/assets/195d5ff3-6037-4586-8748-555513616761)

---

## Step 2: Set Up a Virtual Network and Subnet

- Go to **Virtual Networks** and click *Create*.
- Fill in these details:
  - Name: `AD-VNet`
  - Address space: `10.0.0.0/16`
  - Subnet name: `AD-Subnet`
  - Subnet range: `10.0.1.0/24`
  - Make sure it’s in the same region as your Resource Group.

![Create Virtual Network](https://github.com/user-attachments/assets/7eaa8600-066f-453f-a7e1-a2af0c7fd8de)

---

## Step 3: Create Your Domain Controller VM (`dc-1`)

- Go to **Virtual Machines** > *Create*.
- Set it up like this:
  - Name: `dc-1`
  - Image: Windows Server 2022
  - Username: `labuser`
  - Password: `Cyberlab123!`
  - Region: same as your VNet
  - Size: `Standard_D2s_v3` (2 CPUs, 8 GB RAM)
  - Virtual Network: `AD-VNet`
  - Subnet: `AD-Subnet`
  - Public IP: Enabled
  - Allow RDP inbound on port 3389

![Create DC VM](https://github.com/user-attachments/assets/ed2da35c-813d-4541-ad75-abeda564be89)

---

## Step 4: Lock Down the Domain Controller’s IP

- Once the VM’s up, open **dc-1** in the Azure Portal.
- Go to **Networking** > **IP Configurations**.
- Change the Private IP allocation from dynamic to **Static**.

![Set Static IP](https://github.com/user-attachments/assets/dbfcb103-40e0-4379-907a-90deb3cb3b5e)

Now, connect via RDP to `dc-1`:

- Open Windows Defender Firewall (`wf.msc`).
- For now, disable the firewall on **Domain**, **Private**, and **Public** profiles.

---

## Step 5: Create Your Client VM (`client-1`)

- Back to **Virtual Machines** > *Create*.
- Configure it like this:
  - Name: `client-1`
  - Image: Windows 10
  - Username: `labuser`
  - Password: `Cyberlab123!`
  - Region: same as `dc-1`
  - Virtual Network: `AD-VNet`
  - Subnet: `AD-Subnet`
  - Public IP: Enabled
  - Allow inbound RDP (port 3389)

![Create Client VM](https://github.com/user-attachments/assets/4eb2a1fc-4ba9-4ca6-a3f0-97768b26d870)

---

## Step 6: Point Client DNS to Your Domain Controller

- In the Azure Portal, open **client-1**.
- Go to **Networking** > **Network Settings** > **DNS Servers**.
- Change from **Default** to **Custom**.
- Enter the private IP address of `dc-1`.
- Hit **Save**, then **restart client-1**.

![Configure DNS](https://github.com/user-attachments/assets/d7e466ce-df12-4d54-9105-0f5d99c728d4)

---

## Step 7: Quick Connectivity Check with PowerShell

RDP into `client-1` and open PowerShell:

- Ping the Domain Controller’s private IP:

<p align="center">
  <img width="151" height="49" alt="Screenshot 2025-10-02 092422" src="https://github.com/user-attachments/assets/e70b7473-c7ad-4688-adc7-b4943c12b354" />

</p>

- Run `ipconfig /all` to verify the DNS server is correctly pointing to `dc-1`.

<div style="text-align: center;">
  <img width="114" height="45" alt="Screenshot 2025-10-02 092534" src="https://github.com/user-attachments/assets/1af3df0f-5ece-47c4-aaaa-3d281acbc7f7" />
</div>
<div style="text-align: center; margin-top: 10px;">
  <img width="827" height="499" alt="Screenshot 2025-10-02 092636" src="https://github.com/user-attachments/assets/9455afc7-6d8b-4f55-b432-f6749de3f324" />
</div>


---

## Step 8: Promote `dc-1` to Domain Controller

- Connect to `dc-1` via RDP.
- Open **Server Manager**.
- Click the **flag icon** in the top-right for post-installation tasks.
- Select **Promote this server to a domain controller**.
- In the wizard:
  - Choose **Add a new forest**
  - Domain name: `mydomain.com`
  - Set the DSRM password
  - Complete the wizard and restart

<p align="center">
  <img src="https://github.com/user-attachments/assets/0b5ab5ad-b53c-41da-9c49-f039a9dabf0e" alt="Promote to Domain Controller wizard" width="756" height="533" />
</p>

- After restart, log in as: `mydomain.com\reggie-admin`

<p align="center">
  <img src="https://github.com/user-attachments/assets/b2c4f156-4b50-4e63-b0da-bc37a1b3ce4e" alt="Domain Admin login" width="798" height="476" />
</p>

- Open **Active Directory Users and Computers** to verify setup

<p align="center">
  <img src="https://github.com/user-attachments/assets/dbf5c75f-b67e-4df0-8c55-8a2472d6abc3" alt="Active Directory Users and Computers console" width="822" height="494" />
</p>

---

## Step 9: Join `client-1` to the Domain

- Log into `client-1` as `labuser`
- Right-click Start > **System** > **Rename this PC (Advanced)**

<p align="center">
  <img src="https://github.com/user-attachments/assets/360c7b0f-3916-4639-b8b5-7aa10920b994" alt="System properties rename PC dialog" width="825" height="552" />
</p>

- Under **Computer Name**, click **Change**
- Select **Domain**, type `mydomain.com`
- Enter domain credentials: `mydomain.com\jane_admin`

<p align="center">
  <img src="https://github.com/user-attachments/assets/1fa62d0e-fadb-44b2-af2c-c5129f8c551f" alt="Domain join dialog" width="822" height="494" />
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/a567000d-9c41-4f16-8fab-48d7194a8882" alt="Domain join credentials prompt" width="462" height="304" />
</p>

- After domain join, restart `client-1`

---

## Step 10: Verify `client-1` in Active Directory

- Log into `dc-1` as `mydomain.com\reggie-admin`
- Open **Active Directory Users and Computers**

<p align="center">
  <img src="https://github.com/user-attachments/assets/f618c498-f456-4e22-94fc-3ed4d4ede254" alt="Active Directory Users and Computers console" width="827" height="497" />
</p>

- Navigate to the **Computers** container
- Confirm `client-1` appears

<p align="center">
  <img src="https://github.com/user-attachments/assets/a7df77d9-65e6-4e42-8edf-dcddaa63a371" alt="Client-1 computer object in AD" width="460" height="256" />
</p>

---

## 🧾 Conclusion

You’ve successfully deployed Active Directory in Azure using two virtual machines. The domain controller (`dc-1`) was promoted to manage `mydomain.com`, and the client (`client-1`) was joined to the domain. This setup gives you a working AD environment for testing, learning, or building out more advanced features.

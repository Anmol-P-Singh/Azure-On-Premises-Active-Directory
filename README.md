# 🏗️ How to Set Up Active Directory in Microsoft Azure

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
  - Password: `Cyberlab123!` (make sure to pick your own strong password in real setups)
  - Region: same as your VNet
  - Size: `Standard_D2s_v3` (2 CPUs, 8 GB RAM)
  - Virtual Network: `AD-VNet`
  - Subnet: `AD-Subnet`
  - Public IP: Enabled (so you can RDP in)
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
- For now (since this is a lab), disable the firewall on **Domain**, **Private**, and **Public** profiles.

---

## Step 5: Create Your Client VM (`client-1`)

- Back to **Virtual Machines** > *Create*.
- Configure it like this:
  - Name: `client-1`
  - Image: Windows 10 (latest)
  - Username: `labuser`
  - Password: `Cyberlab123!`
  - Region: same as `dc-1`
  - Virtual Network: `AD-VNet`
  - Subnet: `AD-Subnet`
  - Public IP: Enabled (for RDP)
  - Allow inbound RDP (port 3389)
This VM will act as the machine you want to join to the domain.


![Create Client VM](https://github.com/user-attachments/assets/4eb2a1fc-4ba9-4ca6-a3f0-97768b26d870)

---

## Step 6: Point Client DNS to Your Domain Controller

- In the Azure Portal, open **client-1**.
- Go to **Networking** > **Network Settings** > **DNS Servers**.
- Change from **Default** to **Custom**.
- Enter the private IP address of `dc-1`.
- Hit **Save**, then **restart client-1** for the change to take effect.

![Configure DNS](https://github.com/user-attachments/assets/d7e466ce-df12-4d54-9105-0f5d99c728d4)

---

## Step 7: Quick Connectivity Check with PowerShell

RDP into `client-1` and open PowerShell:

- Ping the Domain Controller’s private IP (like `10.0.0.4`) to check connectivity:

<p align="center">
  <img width="675" height="398" alt="PowerShell Ping" src="https://github.com/user-attachments/assets/c0601cac-60f3-43e6-be2b-af1f5b9fb023" />
</p>

- Run `ipconfig /all` and look for the DNS server entry — it should show the Domain Controller’s IP.

<p align="center">
  <img width="1996" height="1249" alt="PowerShell ipconfig" src="https://github.com/user-attachments/assets/9578cc37-2f34-4305-83b6-2ed99166245b" />
</p>

---

**You now have the basics in place. From here, you can promote your Domain Controller, join the client to the domain, and explore Active Directory features like user and group management.**

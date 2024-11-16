# Azure AD Connect Installation and Configuration on Windows Server 2022

This guide provides a step-by-step process to install and configure **Azure AD Connect** on Windows Server 2022 to synchronize your on-premises Active Directory with Azure AD.

---

## Prerequisites

### System Requirements
- **Operating System**: Windows Server 2016 or later (Server 2022 recommended).
- **Hardware**: At least 4 GB of RAM and a dual-core CPU.
- **Software**: .NET Framework 4.7.2 or later installed.

### Account Permissions
- **Azure AD**: Global Administrator account.
- **On-Premises AD**: Enterprise Admin account.

### Network and DNS
- Ensure DNS resolution for on-premises and Azure AD domains (recommended).
- For this demo, am using just my local domain, it is not revolving but the procedure is still the same.

### Download
- [Azure AD Connect Download](https://www.microsoft.com/en-us/download/details.aspx?id=47594)

---

## Installation Steps

### Step 1: Prepare the Server
1. **Join the Server to the Domain**:
   Ensure the server is part of your on-premises AD domain.
2. **Install Required Features**:
   Open PowerShell as Administrator and run:
   
   ```powershell
   Install-WindowsFeature RSAT-ADDS
   ```

![image](https://github.com/user-attachments/assets/b72e194e-c85a-4ff4-9139-6f31357e091a)


3. **Verify Time Settings**:
   Ensure the server’s time zone matches your domain controllers.

---

### Step 2: Install Azure AD Connect
1. **Run the Installer**:
   Download and run the **`AzureADConnect.msi`** file.
  - If you see something like on this screenshot, click on **`Exit`** and then solve it following these step.

![image](https://github.com/user-attachments/assets/8d6d0906-55a2-4752-99c1-9ac665a5d36a)

### Force .NET Framework to Use TLS 1.2 (mae sure dotnet is already installed)
- Run the following commands in PowerShell with administrative privileges:

```bah

   # Enable TLS 1.2 for .NET Framework (System-wide settings)
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -Name 'SchUseStrongCrypto' -Value 1
Set-ItemProperty -Path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -Name 'SchUseStrongCrypto' -Value 1

   # Verify the settings
Get-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' | Select-Object SchUseStrongCrypto
Get-ItemProperty -Path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' | Select-Object SchUseStrongCrypto

```


![image](https://github.com/user-attachments/assets/5a06f478-4c70-4c79-b6a3-5496e912cf1c)


- Restart your system to apply the changes and run this command in PowerShell again:

```bash
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

```
- Verify the result by running:
```bash
[Net.ServicePointManager]::SecurityProtocol

```
It should now include **`Tls12`**.


![image](https://github.com/user-attachments/assets/ea4229c4-2512-4236-b784-33198ada9f21)

- Now you can continue with the installation. Accept licence terms and conditions by checking **`I agree ......`** and then  click on **`Continue`**.


![image](https://github.com/user-attachments/assets/af5d5427-47a0-4339-8231-31dec84af1f4)


3. **Choose Installation Type**:
   - **Express Settings**: Quick setup with default settings.
   - **Customize**: For advanced configuration (recommended).
   - Here we will go with **`Customize`**.


![image](https://github.com/user-attachments/assets/85fa1b41-184e-436d-bfdc-e5e679eaa48f)



   - Install **synchronization services**.


![image](https://github.com/user-attachments/assets/7cb7d7fb-75de-4350-8b84-62ef00f3503d)


   - Choose **Password Hash Synchronization** (default) or other authentication methods like **Pass-through Authentication** or **Federation**.Then clcik on **`Next`** to continue.


![image](https://github.com/user-attachments/assets/9fa5b537-5335-49db-9793-c5eb401c5da8)

---

### Step 3: Configure Azure AD Connect
1. **Connect to Azure AD**:
   - Enter your **Azure AD Global Administrator** credentials

![image](https://github.com/user-attachments/assets/800d350f-723d-4c81-8f81-aae7101a1516)

In case you encounter this problem, that is how to solve it.

![image](https://github.com/user-attachments/assets/76df7c4e-40b1-4eaf-bc11-9a5fd498fc81)

### Check Group Policy (for servers with strict policies)

- If you're on a domain-joined system, JavaScript might be disabled via Group Policy.
   . Open Group Policy Editor:
```bash 
        Press Win + R, type gpedit.msc, and press Enter.
```
   . Navigate to: **`User Configuration > Administrative Templates > Windows Components > Internet Explorer > Internet Control Panel > Security Page > Internet Zone`**

   . Look for Turn on **`Allow active Scripting`**, set it to **`Enabled`**. Ensure the option Enable is selected.



![image](https://github.com/user-attachments/assets/0d0eaad5-6561-4976-bff5-b35a763811f9)

    
2. **Connect to On-Premises AD**:
   - Provide **Enterprise Admin** credentials for your domain.
3. **Choose Synchronization Options**:
   - Sync all users/groups or filter by **Organizational Units (OUs)** or custom filters.
4. **Enable Optional Features**:
   - **Password Writeback**: Sync password resets from Azure AD to on-premises AD.
   - **Group Writeback**: Sync Azure AD groups to on-premises AD.
   - **Device Writeback**: Enable **Hybrid Azure AD Join**.

---

### Step 4: Complete the Installation
1. **Review and Install**:
   - Verify your settings and click **Install**.
2. **Initial Synchronization**:
   - The **Synchronization Service Manager** will open, and the first sync will start automatically.
3. **Verify in Azure Portal**:
   - Navigate to **Azure Active Directory > Users** and ensure the on-premises users are listed.

---

## Post-Installation Configuration

### Verify Synchronization
1. Open **Synchronization Service Manager**.
2. Check for any synchronization errors.

### Trigger Manual Synchronization
To manually trigger synchronization, open PowerShell as Administrator and run:
```powershell
Start-ADSyncSyncCycle -PolicyType Delta
```
Use `-PolicyType Initial` for a full sync.

### Enable Hybrid Azure AD Join (Optional)
1. Open Azure AD Connect and configure **Device Options**.
2. Enable **Hybrid Azure AD Join** for your domain.

---

## Best Practices
- **Backup Configuration**:
  Regularly back up your Azure AD Connect configuration.
- **Monitor Synchronization**:
  Periodically check synchronization logs for errors.
- **Enable Staging Mode**:
  Configure a second Azure AD Connect server in staging mode as a backup.

---

## Resources
- [Azure AD Connect Documentation](https://learn.microsoft.com/en-us/azure/active-directory/hybrid/whatis-azure-ad-connect)
- [Azure AD Connect Troubleshooting](https://learn.microsoft.com/en-us/azure/active-directory/hybrid/tshoot-connect-sync-errors)

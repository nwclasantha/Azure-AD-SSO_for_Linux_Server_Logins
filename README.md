## Integrating Azure AD SSO for Linux Server Logins Using PuTTY

### Introduction

In enterprise environments, managing user access across multiple servers can become complex, especially when users need to remember unique credentials for each system. Integrating Azure Active Directory (Azure AD) for Single Sign-On (SSO) allows centralized identity management and secure access control, enabling users to log into Linux servers using their existing Azure AD credentials. This setup reduces the administrative burden on IT teams and enhances security by applying Azure AD’s comprehensive identity protection features, such as multi-factor authentication (MFA) and conditional access.

### Objectives

This guide aims to provide a step-by-step approach to:
1. Set up Azure AD-based authentication on Linux servers for centralized, streamlined access control.
2. Configure Linux servers to accept Azure AD SSO, allowing users to log in using their AD credentials.
3. Enable connection via PuTTY, facilitating secure and simplified access for Windows users.
4. Implement secure, scalable, and user-friendly authentication across multiple Linux servers.

### Security Requirements

Implementing Azure AD SSO for Linux servers requires certain security measures and prerequisites to ensure compliance with best practices and safeguard user credentials. Key requirements include:

1. **Azure AD Premium P1 or P2 License**: Required to support SSO and certificate-based authentication for Linux.
2. **SSH Certificate-Based Authentication**: Configuring Azure AD for SSH certificates enables user identities to be verified through certificates rather than traditional passwords.
3. **SSSD Configuration**: `sssd` (System Security Services Daemon) will be used to integrate with Azure AD, managing authorized keys and access control.
4. **Multi-Factor Authentication (MFA)**: Azure AD’s MFA is recommended to secure login attempts, adding an extra layer of security.
5. **Access Control and Permissions**: Ensure that permissions and access are restricted based on the principle of least privilege, limiting access to only necessary users.
6. **Firewall and Network Security**: SSH ports should be configured to allow access only from trusted networks, ensuring secure communication between the client machine and servers.

Here's a complete guide to integrate Azure AD SSO for multiple Linux server logins and connect using PuTTY:

---

## Integrating Azure AD SSO for Linux Server Logins and Connecting via PuTTY

This setup allows Azure AD users to log in to Linux servers using their AD credentials, enabling centralized identity management.

### Prerequisites
1. **Azure AD Premium P1 or P2**: Required for Azure AD-based SSO.
2. **Azure AD SSH Extension**: Enables Linux VM to validate Azure AD users.
3. **Linux VM setup**: Ensure all Linux servers are accessible with administrative access.

---

### Step 1: Install Required Packages on Each Linux Server

1. **Install SSSD and realmd**:
   - On **Debian/Ubuntu** systems:
     ```bash
     sudo apt update
     sudo apt install sssd sssd-tools realmd adcli -y
     ```
   - On **RHEL/CentOS** systems:
     ```bash
     sudo yum install sssd sssd-tools realmd adcli -y
     ```

2. **Install Azure CLI**:
   ```bash
   curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
   ```

---

### Step 2: Configure Azure AD SSO for SSH Login

1. **Log into Azure CLI**:
   ```bash
   az login
   ```

2. **Add Azure AD SSH Extension**:
   ```bash
   az extension add --name azure-ad
   ```

---

### Step 3: Create and Configure an Azure AD Application

1. **Register a New Application in Azure AD**:
   - Go to [Azure AD App registrations](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps).
   - Click **New registration** > enter a name > select **Accounts in this organizational directory only**.
   - Register the application.

2. **Configure API Permissions**:
   - Add **Microsoft Graph** API permissions:
     - `User.Read`
     - `Directory.Read.All`
     - `User.ReadBasic.All`
   - Grant admin consent.

3. **Generate a Client Secret**:
   - Go to **Certificates & Secrets** > **New client secret**.
   - Name it and set an expiration date.
   - Save the generated secret; you will need it in SSSD configuration.

4. **Configure SSH Certificate-Based Authentication**:
   - Generate or use an existing SSH certificate.
   - Associate it with the application in **Certificates & Secrets**.

---

### Step 4: Configure Linux Servers to Use Azure AD for Authentication

1. **Modify SSH Configuration**:
   - Edit SSH daemon configuration:
     ```bash
     sudo nano /etc/ssh/sshd_config
     ```
   - Add the following lines:
     ```config
     AuthorizedKeysCommand /usr/bin/sss_ssh_authorizedkeys
     AuthorizedKeysCommandUser nobody
     ```
   - Restart SSH:
     ```bash
     sudo systemctl restart sshd
     ```

---

### Step 5: Configure SSSD to Use Azure AD

1. **Edit SSSD Configuration**:
   - Open SSSD config:
     ```bash
     sudo nano /etc/sssd/sssd.conf
     ```
   - Add the following:
     ```ini
     [sssd]
     services = nss, pam, ssh
     config_file_version = 2
     domains = your_domain.onmicrosoft.com

     [domain/your_domain.onmicrosoft.com]
     id_provider = ad
     access_provider = ad
     ad_domain = your_domain.onmicrosoft.com
     ad_server = <AD Server IP or hostname>
     ldap_id_mapping = True
     use_fully_qualified_names = False
     default_shell = /bin/bash
     fallback_homedir = /home/%u
     ```
   - Replace placeholders with actual domain details.

2. **Secure SSSD Config**:
   ```bash
   sudo chmod 600 /etc/sssd/sssd.conf
   sudo chown root:root /etc/sssd/sssd.conf
   ```

3. **Enable and Start SSSD**:
   ```bash
   sudo systemctl enable sssd
   sudo systemctl start sssd
   ```

---

### Step 6: Configure SSH Key-Based Authentication for Azure AD Users

1. **Generate an SSH Key** (if needed):
   ```bash
   ssh-keygen -t rsa -b 4096
   ```

2. **Add Public Key to Azure AD**:
   - Go to **Azure AD Portal** > **Users** > **Authentication methods**.
   - Add your SSH public key.

---

### Step 7: Test SSH Login

1. **SSH to the Linux Server**:
   ```bash
   ssh 'your_azure_username@yourdomain.onmicrosoft.com'@server_ip_address
   ```
2. **Troubleshoot**:
   - Check `sssd` status:
     ```bash
     sudo systemctl status sssd
     ```
   - Check logs in `/var/log/sssd/` if needed.

---

### Step 8: Connect to Linux Server Using PuTTY

If using PuTTY on Windows, follow these steps to connect.

1. **Convert SSH Private Key to PuTTY Format (Optional)**:
   - Open **PuTTYgen**.
   - Load your private key file (`id_rsa`).
   - Click **Save private key** and save as `.ppk`.

2. **Configure PuTTY**:
   - Open PuTTY and go to **Session**.
   - Enter the hostname as:
     ```plaintext
     'your_azure_username@yourdomain.onmicrosoft.com'@server_ip_address
     ```
   - In **Connection > SSH > Auth**, select the `.ppk` private key file.
   - Save the session (optional), then click **Open**.

3. **Connect**:
   - Accept the server's SSH key if prompted.
   - Enter your Azure AD credentials if prompted.

If everything is configured correctly, you will connect to the Linux server using Azure AD-based SSH login through PuTTY.

### Step-by-Step Guide

Follow the detailed setup and configuration steps outlined below to integrate Azure AD SSO for multiple Linux servers, concluding with instructions on accessing the servers using PuTTY.

1. **Install Required Packages** on each Linux server to support SSSD, SSH configuration, and Azure CLI.
2. **Configure Azure AD SSO for SSH Login**: Authenticate with the Azure CLI and install any necessary extensions for Azure AD support.
3. **Create and Configure an Azure AD Application**: Register an application within Azure AD for SSH login, configure API permissions, and generate a client secret.
4. **Configure Linux Servers to Use Azure AD**: Update SSH configurations to utilize Azure AD authentication and configure SSSD.
5. **Configure SSH Key-Based Authentication** for Azure AD users by adding public SSH keys to Azure AD user accounts.
6. **Test SSH Login** to verify that Azure AD users can authenticate.
7. **Configure PuTTY Access** for Windows-based access to the Linux servers.

---

### Conclusion

By integrating Azure AD for SSO with Linux servers, organizations benefit from centralized user management, enhanced security controls, and simplified access processes. This approach leverages Azure AD's robust security features, such as MFA and conditional access policies, ensuring secure and controlled access across Linux environments. Additionally, allowing access via PuTTY facilitates a seamless experience for users on Windows, making Azure AD-based SSO an accessible and secure solution for cross-platform environments. This solution effectively enhances security, reduces administrative complexity, and supports a streamlined and scalable approach to Linux server management.

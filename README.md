<p align="center">
<img src="https://i.imgur.com/Clzj7Xs.png" alt="osTicket logo"/>
</p>

<h1>osTicket - Prerequisites and Installation</h1>
This tutorial outlines the prerequisites and installation of the open-source help desk ticketing system osTicket.<br />



<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Internet Information Services (IIS)

<h2>Operating Systems Used </h2>

- Windows 10</b> (21H2)

<h2>List of Prerequisites</h2>

- Microsoft Azure account- to create and manage the virtual machine
- Virtual Machine(Windows 10 21H2)-where osTicket will be installed
- Remote Desktop Connection-to access your virtual machine
- Internte Information Services-to access your virtual machine
- PHP & MySQL-required to support the osTicket platform

<h2>Installation Steps</h2>

# osTicket: Prerequisites and Installation (Azure Step-by-Step)

This guide walks you step by step through deploying osTicket on an Azure Ubuntu VM using the LAMP stack.

---

## Prerequisites

### 1. Azure Subscription
- Ensure you have an active [Azure subscription](https://portal.azure.com/).

### 2. Create an Ubuntu VM
1. Go to the [Azure Portal](https://portal.azure.com/).
2. Click **Create a resource** > **Virtual Machine**.
3. Choose:
    - **Image:** Ubuntu 22.04 LTS
    - **Size:** At least Standard B1s (1 vCPU, 1 GiB RAM)
    - **Authentication:** SSH (recommended)
    - **Inbound ports:** Select **HTTP (80)**, **SSH (22)**, and **HTTPS (443)** if planning to use SSL.
4. Review + Create, then launch the VM.

---

## Installation

### 1. Connect to Your VM

```sh
ssh azureuser@<YOUR_VM_PUBLIC_IP>
```
(Replace `azureuser` and `<YOUR_VM_PUBLIC_IP>` with your actual username and public IP.)

---

### 2. Update System Packages

```sh
sudo apt update && sudo apt upgrade -y
```

---

### 3. Install Apache Web Server

```sh
sudo apt install -y apache2
```
Check Apache is running:
```sh
sudo systemctl status apache2
```
(Press `q` to exit status.)

---

### 4. Install MySQL Database Server

```sh
sudo apt install -y mysql-server
```
Secure MySQL:
```sh
sudo mysql_secure_installation
```
Follow prompts (set root password, remove anonymous users, disallow root remote login, remove test DB, reload privileges).

---

### 5. Install PHP and Required Extensions

```sh
sudo apt install -y php php-mysql php-imap php-gd php-xml php-mbstring php-intl php-apcu unzip
```
Check PHP version:
```sh
php -v
```

---

### 6. Create osTicket Database and User

```sh
sudo mysql -u root -p
```
In the MySQL shell, run:

```sql
CREATE DATABASE osticket;
CREATE USER 'osticketuser'@'localhost' IDENTIFIED BY 'StrongPassword123!';
GRANT ALL PRIVILEGES ON osticket.* TO 'osticketuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

### 7. Download osTicket

```sh
cd /tmp
curl -LO https://github.com/osTicket/osTicket/releases/download/v1.18.1/osTicket-v1.18.1.zip
unzip osTicket-v1.18.1.zip
sudo mv upload /var/www/html/osticket
```

---

### 8. Set Directory Permissions

```sh
sudo chown -R www-data:www-data /var/www/html/osticket
sudo chmod -R 755 /var/www/html/osticket
```

---

### 9. Configure Apache for osTicket

Create a new virtual host file:

```sh
sudo nano /etc/apache2/sites-available/osticket.conf
```

Paste the following (update `ServerName` as needed):

```apache
<VirtualHost *:80>
    ServerAdmin admin@example.com
    DocumentRoot /var/www/html/osticket
    ServerName <YOUR_DOMAIN_OR_VM_IP>

    <Directory /var/www/html/osticket>
        Options FollowSymlinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/osticket_error.log
    CustomLog ${APACHE_LOG_DIR}/osticket_access.log combined
</VirtualHost>
```

Enable the site and rewrite module, then reload Apache:

```sh
sudo a2ensite osticket
sudo a2enmod rewrite
sudo systemctl reload apache2
```

---

### 10. Allow HTTP/HTTPS Traffic in Azure

- Go to your VM in the Azure Portal.
- Under **Networking** > **Inbound port rules**, ensure **HTTP (80)** and **HTTPS (443)** are allowed.

---

### 11. Complete osTicket Web Installer

1. Open a browser and go to:  
   `http://<YOUR_VM_PUBLIC_IP>/osticket`
2. Follow the web installer:
    - **Database Name:** osticket
    - **MySQL Username:** osticketuser
    - **Password:** StrongPassword123!
    - Set admin details (name, email, password)

---

### 12. Post-installation Steps

```sh
sudo chmod 0644 /var/www/html/osticket/include/ost-config.php
sudo rm -rf /var/www/html/osticket/setup/
```

---

### 13. (Optional) Enable SSL with Certbot

1. Install Certbot:
    ```sh
    sudo apt install -y certbot python3-certbot-apache
    ```
2. Obtain and install SSL certificate:
    ```sh
    sudo certbot --apache
    ```

---

## You’re Done!

osTicket is now installed and running on your Azure VM.  

---

**References:**
- [osTicket Docs](https://docs.osticket.com/)
- [Azure Linux VM Quickstart](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-portal)
- [osTicket GitHub Releases](https://github.com/osTicket/osTicket/releases)


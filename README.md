# 🚀 Nextcloud Deployment on AWS EC2 (Ubuntu 22.04 + LAMP + LVM Storage + Custom AWS VPC)

![Nextcloud Logo](https://upload.wikimedia.org/wikipedia/commons/6/60/Nextcloud_Logo.svg)

## 📌 Overview
This project showcases the **end-to-end deployment of Nextcloud** on an **AWS EC2 Ubuntu 22.04 instance** using the **LAMP stack** (Linux, Apache, MySQL, PHP).  
The setup also features **LVM-based persistent storage** mounted at `/storage` for scalability and a **Custom AWS VPC** for enhanced network isolation and security.

---

## 🛠️ Features
- 🛡 **Custom AWS VPC** for secure, isolated networking.
- ☁ **Nextcloud** for private file storage and collaboration.
- ⚙ **LAMP Stack** (Apache, MySQL/MariaDB, PHP 8.1).
- 💾 **LVM-based persistent storage** at `/storage`.
- 🔐 **Security Groups** for access control.
- 🌍 **Trusted domain configuration** for public access.
- 📸 **Screenshots & visual setup guide**.

---

## 🌐 AWS VPC Setup
A **Custom AWS VPC** was created instead of using the default one to have full control over IP addressing, routing, and security.

### Steps:
1. **Create VPC** → CIDR block: `10.0.0.0/16`.
2. **Add Subnets**:
   - Public Subnet → `10.0.1.0/24`.
   - Private Subnet → optional (for DB isolation).
3. **Internet Gateway** → attach to VPC.
4. **Route Table** → public subnet routes `0.0.0.0/0` → Internet Gateway.
5. **Launch EC2** → inside public subnet with Elastic IP.
6. **Security Groups** → allow:
   - SSH (22)
   - HTTP (80)
   - HTTPS (443)

✅ **Benefits**:
- Full network isolation.
- Custom IP addressing.
- Secure integration with future AWS resources.

---

## 🚀 Deployment Steps

### 1️⃣ Launch EC2 in Custom VPC
- OS: **Ubuntu 22.04 LTS**
- Instance type: t2.micro (free tier friendly)
- Storage: Root + EBS volume for LVM

### 2️⃣ Install LAMP Stack
\`\`\`bash
sudo apt update && sudo apt upgrade -y
sudo apt install apache2 mysql-server php libapache2-mod-php php-mysql php-xml php-zip php-curl php-mbstring php-gd php-intl unzip -y
\`\`\`

### 3️⃣ Configure LVM Storage
\`\`\`bash
sudo pvcreate /dev/xvdf
sudo vgcreate nxtcld /dev/xvdf
sudo lvcreate -l 100%FREE -n lvol0 nxtcld
sudo mkfs.ext4 /dev/nxtcld/lvol0
sudo mkdir /storage
echo '/dev/nxtcld/lvol0 /storage ext4 defaults 0 2' | sudo tee -a /etc/fstab
sudo mount -a
\`\`\`

### 4️⃣ Install Nextcloud
\`\`\`bash
wget https://download.nextcloud.com/server/releases/nextcloud-27.1.0.zip
unzip nextcloud-27.1.0.zip
sudo mv nextcloud /var/www/html/
sudo chown -R www-data:www-data /var/www/html/nextcloud
sudo chmod -R 755 /var/www/html/nextcloud
\`\`\`

### 5️⃣ Configure Apache
\`\`\`apache
<VirtualHost *:80>
    ServerAdmin admin@example.com
    DocumentRoot /var/www/html/nextcloud
    ServerName your_domain_or_ip

    <Directory /var/www/html/nextcloud>
        Require all granted
        AllowOverride All
        Options FollowSymLinks MultiViews
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
\`\`\`
\`\`\`bash
sudo a2ensite nextcloud.conf
sudo a2enmod rewrite headers env dir mime
sudo systemctl restart apache2
\`\`\`

### 6️⃣ Fix “Untrusted Domain” Error
\`\`\`bash
sudo nano /var/www/html/nextcloud/config/config.php
\`\`\`
Add:
\`\`\`php
'trusted_domains' =>
  array (
    0 => 'your-ec2-public-ip',
    1 => 'your-domain.com',
  ),
\`\`\`

---

## 📸 Screenshots

---

## 📜 License
Licensed under the **MIT License** – feel free to use and modify.

---
💡 *This project is a complete beginner-to-intermediate guide for deploying a secure, production-ready Nextcloud instance on AWS.*

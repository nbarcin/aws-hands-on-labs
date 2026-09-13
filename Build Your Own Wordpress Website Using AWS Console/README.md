# Deploy WordPress on Amazon EC2

A hands-on AWS project demonstrating how to deploy a **WordPress website on an Amazon EC2 instance** using Amazon Linux 2023, Apache, PHP, PHP-FPM, and MariaDB.

This project focuses on practical **AWS cloud, Linux administration, web server configuration, database management, and application deployment** skills.

---

## 🎯 Objectives

In this lab, I:

* Launched an Amazon EC2 instance
* Configured an EC2 Security Group
* Connected to the server using SSH
* Installed and configured Apache
* Installed PHP and PHP-FPM
* Installed and configured MariaDB
* Secured the MariaDB installation
* Created a dedicated WordPress database and user
* Installed phpMyAdmin
* Downloaded and configured WordPress
* Configured `wp-config.php`
* Configured Apache for WordPress permalinks
* Deployed and accessed a working WordPress website

---

## 🏗️ Architecture

```text
                         Internet
                            │
                            │ HTTP :80
                            ▼
                  ┌─────────────────────┐
                  │   EC2 Security      │
                  │       Group         │
                  │                     │
                  │  SSH :22  My IP    │
                  │  HTTP :80  Public   │
                  └──────────┬──────────┘
                             │
                             ▼
              ┌──────────────────────────────┐
              │      Amazon EC2              │
              │      Amazon Linux 2023       │
              │                              │
              │  ┌────────────────────────┐  │
              │  │ Apache (httpd)          │  │
              │  └────────────┬───────────┘  │
              │               │              │
              │  ┌────────────▼───────────┐  │
              │  │ PHP / PHP-FPM           │  │
              │  └────────────┬───────────┘  │
              │               │              │
              │  ┌────────────▼───────────┐  │
              │  │ WordPress               │  │
              │  └────────────┬───────────┘  │
              │               │              │
              │  ┌────────────▼───────────┐  │
              │  │ MariaDB                 │  │
              │  │ my-wordpress-db        │  │
              │  └────────────────────────┘  │
              └──────────────────────────────┘
```

---

## ☁️ AWS Services

* **Amazon EC2** — compute server
* **Amazon VPC** — network environment
* **Security Groups** — instance-level firewall
* **Elastic/Public IPv4** — access to the web server
* **AWS Console** — infrastructure management

---

## 💻 Technologies

| Technology        | Purpose                      |
| ----------------- | ---------------------------- |
| Amazon Linux 2023 | Operating system             |
| Apache            | Web server                   |
| PHP               | Application runtime          |
| PHP-FPM           | PHP process manager          |
| MariaDB           | Relational database          |
| phpMyAdmin        | Database administration      |
| WordPress         | Web application              |
| SSH               | Remote server administration |
| Bash              | Linux administration         |

---

# 1. Launch EC2 Instance

Launch an EC2 instance using **Amazon Linux 2023**.

### Example Configuration

* AMI: Amazon Linux 2023
* Instance type: `t2.micro` / `t3.micro`
* Key pair: SSH key pair
* Security Group:

  * SSH — TCP 22 — My IP
  * HTTP — TCP 80 — `0.0.0.0/0`

For a production environment, HTTPS (443) should also be configured.

---

# 2. Connect to EC2

Connect using SSH:

```bash
chmod 400 your-key.pem

ssh -i your-key.pem ec2-user@PUBLIC-IP
```

Verify the operating system:

```bash
cat /etc/os-release
```

Expected:

```text
Amazon Linux 2023
```

Check architecture:

```bash
uname -m
```

---

# 3. Install Apache, PHP and PHP-FPM

Amazon Linux 2023 uses **DNF** instead of the older `amazon-linux-extras` workflow.

Update the system:

```bash
sudo dnf upgrade -y
```

Install Apache and PHP packages:

```bash
sudo dnf install -y httpd wget php-fpm php-mysqli php-json php php-devel
```

Check PHP:

```bash
php -v
```

Enable and start Apache:

```bash
sudo systemctl enable --now httpd
```

Enable and start PHP-FPM:

```bash
sudo systemctl enable --now php-fpm
```

Check service status:

```bash
sudo systemctl status httpd
sudo systemctl status php-fpm
```

---

# 4. Test Apache

Open a browser and navigate to:

```text
http://PUBLIC-IP
```

The Apache test page should appear.

If the page does not load, verify:

```bash
sudo systemctl status httpd
```

and:

```bash
sudo ss -lntp | grep ':80'
```

Also verify that TCP port 80 is allowed in the EC2 Security Group.

---

# 5. Configure Linux Permissions

Add `ec2-user` to the Apache group:

```bash
sudo usermod -a -G apache ec2-user
```

Log out and reconnect:

```bash
exit
```

Then reconnect using SSH.

Verify group membership:

```bash
groups
```

Configure `/var/www` ownership:

```bash
sudo chown -R ec2-user:apache /var/www
```

Configure directory permissions:

```bash
sudo chmod 2775 /var/www
```

Apply permissions to directories:

```bash
find /var/www -type d -exec sudo chmod 2775 {} \;
```

Apply permissions to files:

```bash
find /var/www -type f -exec sudo chmod 0664 {} \;
```

> These permissions are suitable for a learning lab. Production environments should follow stricter least-privilege practices.

---

# 6. Test PHP

Create a temporary PHP information page:

```bash
echo '<?php phpinfo(); ?>' | sudo tee /var/www/html/phpinfo.php
```

Open:

```text
http://PUBLIC-IP/phpinfo.php
```

If PHP is configured correctly, the PHP information page will appear.

After testing, remove the file:

```bash
sudo rm /var/www/html/phpinfo.php
```

---

# 7. Install MariaDB

Install the MariaDB server:

```bash
sudo dnf install -y mariadb105-server
```

Enable and start MariaDB:

```bash
sudo systemctl enable --now mariadb
```

Check the service:

```bash
sudo systemctl status mariadb
```

Connect to MariaDB:

```bash
sudo mariadb
```

Check the database version:

```sql
SELECT VERSION();
```

---

# 8. Secure MariaDB

Run:

```bash
sudo mariadb-secure-installation
```

If the command is unavailable, try:

```bash
sudo mysql_secure_installation
```

The security process can remove:

* Anonymous users
* Remote root access
* Test databases

It also reloads privilege tables.

---

# 9. Create WordPress Database

Open MariaDB:

```bash
sudo mariadb
```

Create the database:

```sql
CREATE DATABASE `my-wordpress-db`;
```

Create the WordPress database user:

```sql
CREATE USER 'wordpress_user'@'localhost'
IDENTIFIED BY 'YOUR_STRONG_PASSWORD';
```

Grant privileges:

```sql
GRANT ALL PRIVILEGES
ON `my-wordpress-db`.*
TO 'wordpress_user'@'localhost';
```

Apply the changes:

```sql
FLUSH PRIVILEGES;
```

Verify the database:

```sql
SHOW DATABASES;
```

Verify the user's permissions:

```sql
SHOW GRANTS FOR 'wordpress_user'@'localhost';
```

Exit:

```sql
exit;
```

### Important

Because the database name contains hyphens, it must be enclosed in backticks:

```sql
`my-wordpress-db`
```

---

# 10. Install phpMyAdmin

Install additional PHP packages:

```bash
sudo dnf install -y php-mbstring php-xml
```

Restart PHP-FPM and Apache:

```bash
sudo systemctl restart php-fpm
sudo systemctl restart httpd
```

Move to the Apache web directory:

```bash
cd /var/www/html
```

Download phpMyAdmin:

```bash
wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.tar.gz
```

Create the directory:

```bash
mkdir -p phpMyAdmin
```

Extract phpMyAdmin:

```bash
tar -xvzf phpMyAdmin-latest-all-languages.tar.gz \
-C phpMyAdmin --strip-components 1
```

Remove the downloaded archive:

```bash
rm phpMyAdmin-latest-all-languages.tar.gz
```

Check MariaDB:

```bash
sudo systemctl status mariadb
```

Access phpMyAdmin:

```text
http://PUBLIC-IP/phpMyAdmin
```

### Security Note

phpMyAdmin is useful for learning and database administration, but exposing it directly to the public internet is not recommended for a production environment without additional security controls.

---

# 11. Download WordPress

Move to `/var/www`:

```bash
cd /var/www
```

Download WordPress:

```bash
wget https://wordpress.org/latest.tar.gz
```

Extract:

```bash
tar -xzf latest.tar.gz
```

This creates:

```text
/var/www/wordpress
```

---

# 12. Configure WordPress Database

Create the WordPress configuration file:

```bash
cd /var/www
cp wordpress/wp-config-sample.php wordpress/wp-config.php
```

Edit the file:

```bash
nano wordpress/wp-config.php
```

Update the database configuration:

```php
define( 'DB_NAME', 'my-wordpress-db' );
define( 'DB_USER', 'wordpress_user' );
define( 'DB_PASSWORD', 'YOUR_STRONG_PASSWORD' );
define( 'DB_HOST', 'localhost' );
```

WordPress also uses authentication keys and salts for security.

Generate fresh values using the official WordPress secret-key generator and replace the sample values in `wp-config.php`.

Save the file:

```text
Ctrl + O
Enter
Ctrl + X
```

---

# 13. Deploy WordPress to Apache

Create the website directory:

```bash
sudo mkdir -p /var/www/html/mywordpresswebsite
```

Copy WordPress files:

```bash
sudo cp -r wordpress/* /var/www/html/mywordpresswebsite/
```

Verify:

```bash
ls -la /var/www/html/mywordpresswebsite
```

Expected files/directories include:

```text
index.php
wp-admin/
wp-content/
wp-includes/
wp-config.php
wp-login.php
wp-settings.php
```

---

# 14. Configure Apache Permalinks

Edit Apache configuration:

```bash
sudo nano /etc/httpd/conf/httpd.conf
```

Find:

```apache
<Directory "/var/www/html">
```

Inside this section, change:

```apache
AllowOverride None
```

to:

```apache
AllowOverride All
```

This allows WordPress `.htaccess` rules to work.

Test the Apache configuration:

```bash
sudo apachectl configtest
```

Expected:

```text
Syntax OK
```

Restart Apache:

```bash
sudo systemctl restart httpd
```

---

# 15. Verify Services

Check Apache:

```bash
sudo systemctl status httpd
```

Check PHP-FPM:

```bash
sudo systemctl status php-fpm
```

Check MariaDB:

```bash
sudo systemctl status mariadb
```

Enable services at boot:

```bash
sudo systemctl enable httpd
sudo systemctl enable php-fpm
sudo systemctl enable mariadb
```

---

# 16. Complete WordPress Installation

Open:

```text
http://PUBLIC-IP/mywordpresswebsite
```

The WordPress installation page should appear.

Complete the following:

* Site Title
* WordPress Admin Username
* Strong Password
* Email Address

### Important

The WordPress administrator account is **different** from:

* `ec2-user` → Linux user
* `root` → MariaDB administrative user
* `wordpress_user` → WordPress database user

Keep these credentials separate.


# 💡 Key Concepts Learned

### Amazon EC2

Provides virtual servers that can run applications and web services in AWS.

### Security Group

Acts as a stateful virtual firewall controlling inbound and outbound traffic for the EC2 instance.

### Apache

Receives HTTP requests and serves web content.

### PHP / PHP-FPM

PHP executes WordPress application code, while PHP-FPM manages PHP processes.

### MariaDB

Stores WordPress application data such as users, posts, settings, and metadata.

### phpMyAdmin

Provides a web-based interface for managing MySQL/MariaDB databases.

### wp-config.php

Contains WordPress database connection settings and important security configuration.

### AllowOverride All

Allows Apache `.htaccess` rules to be used, which is important for WordPress pretty permalinks.

## ⭐ Project Outcome

Successfully deployed a functional **WordPress website on Amazon EC2** using Amazon Linux 2023 and a LAMP-style architecture.

This project demonstrates practical experience with:

**Cloud Infrastructure → Linux → Networking → Web Servers → Databases → Application Deployment → Security**

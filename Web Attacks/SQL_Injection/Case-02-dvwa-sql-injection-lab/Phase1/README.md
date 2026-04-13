# 🔴 Phase 1: Environment Setup – DVWA SQL Injection Lab

## 📌 Overview

This phase focuses on setting up a vulnerable web application environment to simulate real-world SQL Injection attacks. The objective is to deploy a fully functional LAMP-based web server with DVWA and ensure it is accessible from an attacker machine.

---

## 🎯 Objectives

* Install and configure Ubuntu Server
* Set up LAMP stack (Apache, MySQL, PHP)
* Deploy DVWA application
* Configure database connectivity
* Ensure application accessibility over network

---

## 🧰 Technologies Used

| Component      | Technology    |
| -------------- | ------------- |
| OS             | Ubuntu Server |
| Web Server     | Apache2       |
| Database       | MySQL         |
| Backend        | PHP           |
| Vulnerable App | DVWA          |

---

## ⚙️ Step-by-Step Setup

---

### 🖥️ 1. Ubuntu Server Installation

* Installed Ubuntu Server on VirtualBox
* Allocated minimal resources (2GB RAM, 2 CPU cores)
* Enabled OpenSSH for remote access

---

### 🌐 2. Network Configuration

* Changed network mode from **NAT → Bridged Adapter**
* Assigned IP via DHCP (e.g., `192.168.x.x`)
* Verified connectivity using:

  ```bash
  ping <ubuntu-ip>
  ```

---

### 🧱 3. LAMP Stack Installation

Installed required packages:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install apache2 mysql-server php php-mysqli php-gd libapache2-mod-php git -y
```

---

### 🔧 4. Service Configuration

```bash
sudo systemctl start apache2
sudo systemctl enable apache2

sudo systemctl start mysql
sudo systemctl enable mysql
```

---

### 📦 5. DVWA Deployment

```bash
cd /var/www/html
sudo git clone https://github.com/digininja/DVWA.git dvwa
```

---

### 🔐 6. Permissions Setup

```bash
sudo chown -R www-data:www-data /var/www/html/dvwa
sudo chmod -R 755 /var/www/html/dvwa
```

---

### 🗄️ 7. Database Configuration

```bash
sudo mysql
```

```sql
CREATE DATABASE dvwa;
CREATE USER 'dvwa'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON dvwa.* TO 'dvwa'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

### ⚙️ 8. DVWA Configuration

```bash
cd /var/www/html/dvwa/config
sudo cp config.inc.php.dist config.inc.php
sudo nano config.inc.php
```

Updated:

```php
$_DVWA['db_user'] = 'dvwa';
$_DVWA['db_password'] = 'password';
$_DVWA['db_database'] = 'dvwa';
```

---

### 🔁 9. Restart Services

```bash
sudo systemctl restart apache2
sudo systemctl restart mysql
```

---

### 🌐 10. Application Access

Accessed DVWA from attacker machine:

```
http://<ubuntu-ip>/dvwa
```

---

### 🧪 11. Database Initialization

* Opened:

  ```
  /dvwa/setup.php
  ```
* Clicked **Create / Reset Database**

---

## ✅ Validation

* DVWA login page accessible
* Database initialized successfully
* Login with default credentials:

  * admin / password

---

## ⚠️ Challenges Faced & Fixes

---

### ❌ 1. “Not Found” Error (404)

**Cause:**

* Incorrect URL path
* Case sensitivity issue (`DVWA` vs `dvwa`)

**Fix:**

* Use correct lowercase path:

  ```
  /dvwa
  ```

---

### ❌ 2. HTTP 500 Error During Setup

**Cause:**

* MySQL authentication failure
* Mismatch between config file and database credentials

**Fix:**

```sql
ALTER USER 'dvwa'@'localhost' IDENTIFIED BY 'password';
```

---

### ❌ 3. UNION Queries Causing 500 Error

**Cause:**

* Incorrect number of columns in SQL query

**Fix:**

* Used `ORDER BY` to determine column count

---

### ❌ 4. Network Accessibility Issue

**Cause:**

* VM set to NAT mode

**Fix:**

* Changed to **Bridged Adapter**
* Ensured same network as attacker machine

---

### ❌ 5. Apache Serving Default Page Only

**Cause:**

* `index.html` overriding DVWA directory

**Fix:**

```bash
sudo rm /var/www/html/index.html
```

---

### ❌ 6. Database Connection Failure

**Cause:**

* Missing config file

**Fix:**

```bash
cp config.inc.php.dist config.inc.php
```

---

### ❌ 7. PHP-MySQL Module Missing

**Cause:**

* `php-mysqli` not installed

**Fix:**

```bash
sudo apt install php-mysqli
```

---

## 🧠 Key Learnings

* Linux is case-sensitive → impacts URL access
* Database credentials must match application config
* HTTP status codes help identify issue type:

  * 404 → path issue
  * 500 → backend/database issue
* Network configuration is critical in lab setups

---

## 🏁 Conclusion

This phase successfully established a vulnerable web environment capable of simulating SQL Injection attacks. It also highlighted common setup issues and their resolutions, providing a strong foundation for attack and detection phases.

---

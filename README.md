# Client-Server Architecture with MySQL (Project 102)

A robust, enterprise-grade deployment demonstrating a fully functional decoupled two-tier architecture using native **Oracle MySQL 8.4 LTS** hosted on separate **AWS EC2 Linux Virtual Servers**. Communication is confined strictly to the internal AWS private network environment leveraging advanced Security Group scoping.

---

## 🏗️ Architectural Blueprint

```
 ┌──────────────────────────┐               ┌──────────────────────────┐
 │    AWS EC2 Instance      │               │    AWS EC2 Instance      │
 │     "mysql client"       │               │     "mysql server"       │
 ├──────────────────────────┤               ├──────────────────────────┤
 │ MySQL Client Utility CLI │               │ MySQL Daemon (mysqld)    │
 │ Private IP: 172.31.X.X   │               │ Private IP: 172.31.Y.Y   │
 └─────────────┬────────────┘               └─────────────▲────────────┘
               │                                          │
               │        TCP Connection on Port 3306       │
               └──────────────────────────────────────────┘
                         (Restricted Inbound Rule)
```

---

## 🛠️ Phase-by-Phase Deployment Guide

### Phase 1: Server Provisioning (`mysql server`)

#### 1. Repository Pipeline Configuration
Since native Oracle MySQL 8.4 LTS is mandatory, download the community release architecture packages and sync signatures:
```bash
# Add official Oracle community repository configuration
sudo dnf install -y https://dev.mysql.com/get/mysql84-community-release-el9-1.noarch.rpm

# Import official GPG verification keys
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql

# Install full relational server engine
sudo dnf install -y mysql-community-server

# Bootstrap and enable daemon longevity
sudo systemctl enable --now mysqld
```

#### 2. Extract Temporary Key & Secure Server
Locate the automated bootstrap password string to initiate the hardened setup tool:
```bash
sudo grep 'temporary password' /var/log/mysqld.log
```
Run the automated initialization wizard:
```bash
sudo mysql_secure_installation
```
*Provide your custom root password (`113355pasasas$`), terminate public anonymous profiles, and disable root remote execution pathways.*

#### 3. Update Networking Scoping
Modify configuration layout values to intercept incoming private adapter sockets:
```bash
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```
Change the bind pointer:
```ini
bind-address = 0.0.0.0
```
Refresh running thread states:
```bash
sudo systemctl restart mysqld
```

#### 4. Configure Application Dedicated Profile
Establish the secure client network identity workspace mapping:
```sql
CREATE USER 'remote_user'@'%' IDENTIFIED BY 'ProjectPassword123!';
GRANT ALL PRIVILEGES ON *.* TO 'remote_user'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

![Server Configuration Screenshot](images/server_setup.png)

---

### Phase 2: Firewall Security Infrastructure (AWS VPC)

Isolate exposure matrix rules down within your security groups stack pane:

1. Select your target **`mysql server`** instance context wrapper inside AWS EC2 dashboard.
2. Edit **Inbound Rules** routing matrix to reflect strict matching parameters:
   - **Type:** `MySQL/Aurora (TCP)`
   - **Port Range:** `3306`
   - **Source:** `Custom` ──> `<mysql_client_private_ip>/32` *(or the explicit Client Security Group ID)*

![AWS Security Group Rules Screenshot](images/aws_security_group.png)

---

### Phase 3: Client Deployment & Handshake (`mysql client`)

#### 1. Command-Line Utility Stack Provisioning
Prepare client interface dependencies matching your server version specifications:
```bash
sudo dnf install -y https://dev.mysql.com/get/mysql84-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql
sudo dnf install -y mysql-community-client
```

#### 2. Trigger Client Core Authentication Engine
Run the client tool interface direct connecting through internal VPC routing channels:
```bash
mysql -h <server_private_ip> -u remote_user -p
```
*Provide credential mapping parameters when prompted:* `ProjectPassword123!`

#### 3. Validate Engine Pipeline Matrix
```sql
SHOW DATABASES;
```

![Client Verification Output Screenshot](images/client_verification.png)

---

## ⚡ Troubleshooting & Diagnostic Engine

### 1. Connection Timed Out (`Hanging Terminal`)
- **Reason:** AWS Security Group blocking ingress packages.
- **Fix:** Ensure client IP matches precisely with a `/32` suffix mask on the server's ruleset workspace.

### 2. Connection Refused (`Error 2003`)
- **Reason:** Server socket is bounded only on local localhost looping pathways (`127.0.0.1`).
- **Fix:** Ensure configuration file tracking points to `0.0.0.0` and bounce the daemon.

---
**Author:** Amar Saleem  
*Systems & Infrastructure Operations Analyst*

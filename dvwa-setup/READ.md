DVWA Home Lab Setup — LAMP Stack on Ubuntu 22.04

Overview

This documents the installation and configuration of DVWA (Damn Vulnerable Web Application) on a LAMP stack as part of an ongoing SOC home lab build. DVWA is a deliberately vulnerable PHP/MySQL web application used to practice web application security techniques including SQL injection, XSS, CSRF, file inclusion, and command injection.

This setup runs on the existing Ubuntu Target VM inside an isolated VMware lab network — no new VM required, no public internet exposure.


Environment

ComponentDetailsHost OSUbuntu Desktop 22.04.5 LTSWeb ServerApache 2.4DatabaseMySQL 8.0LanguagePHP 8.1NetworkVMware VMnet1 host-only (isolated lab network)VM RoleUbuntu Target — existing hardened lab machineDVWA Accesshttp://192.168.x.x/dvwa

This machine is part of a larger four-VM SOC lab:


Wazuh SIEM — 192.168.x.x (security monitoring)
Ubuntu Target — 192.168.x.x (this machine — LAMP + DVWA + Wazuh agent + Suricata)
Kali Linux — 192.168.x.x (attacker machine)
Metasploitable2 — 192.168.x.x (additional vulnerable target)



Why LAMP Stack Instead of Docker

Docker would have been faster to set up but LAMP was the deliberate choice for one reason: understanding the full stack.

LAMP stands for Linux, Apache, MySQL, PHP — the actual technology stack running the majority of real-world web applications encountered in bug bounty programs. Installing DVWA on LAMP means configuring the web server, the database, and the application language separately and understanding how they connect. When a SQL injection fires, you understand it is injecting into a MySQL query executed by PHP and served through Apache. That context matters.

Docker abstracts all of that away. LAMP teaches it.


Installation — LAMP Stack

Step 1 — Update the system

bashsudo apt update && sudo apt upgrade -y

Step 2 — Install Apache

bashsudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2

Verify: open a browser and navigate to http://localhost — Apache default page should appear.

Step 3 — Install MySQL

bashsudo apt install mysql-server -y
sudo systemctl enable mysql
sudo systemctl start mysql

Secure the installation:

bashsudo mysql_secure_installation

Step 4 — Install PHP and required extensions

bashsudo apt install php php-mysql php-gd php-xml libapache2-mod-php -y

Step 5 — Restart Apache to load PHP module

bashsudo systemctl restart apache2


Installation — DVWA

Step 1 — Download DVWA

bashcd /var/www/html
sudo git clone https://github.com/digininja/DVWA.git dvwa

Step 2 — Set permissions

bashsudo chown -R www-data:www-data /var/www/html/dvwa
sudo chmod -R 755 /var/www/html/dvwa

Step 3 — Configure DVWA

bashcd /var/www/html/dvwa/config
sudo cp config.inc.php.dist config.inc.php
sudo nano config.inc.php

Set the database credentials:

php$_DVWA['db_user']     = 'dvwa';
$_DVWA['db_password'] = 'dvwapassword';
$_DVWA['db_database'] = 'dvwa';

Step 4 — Create the MySQL database and user

bashsudo mysql -u root

Inside the MySQL prompt:

sqlCREATE DATABASE dvwa;
CREATE USER 'dvwa'@'localhost' IDENTIFIED BY 'dvwapassword';
GRANT ALL PRIVILEGES ON dvwa.* TO 'dvwa'@'localhost';
FLUSH PRIVILEGES;
EXIT;

Step 5 — Configure PHP settings

bashsudo nano /etc/php/8.1/apache2/php.ini

Find and change:

allow_url_fopen = On
allow_url_include = On

Restart Apache:

bashsudo systemctl restart apache2


MySQL 8.0 Compatibility Fix

The Problem

When navigating to http://192.168.x.x/dvwa/setup.php and clicking Create / Reset Database, the page returned a blank white screen. The setup process died silently.

Root cause: DVWA's setup script (/var/www/html/dvwa/includes/DBMS/MySQL.php) contained several ALTER TABLE ... ADD COLUMN IF NOT EXISTS statements:

sqlALTER TABLE users ADD COLUMN IF NOT EXISTS role VARCHAR(20) DEFAULT 'user';
ALTER TABLE users ADD COLUMN IF NOT EXISTS account_enabled TINYINT(1) DEFAULT 1;

The IF NOT EXISTS modifier on ALTER TABLE is a MariaDB extension — it was never standard SQL. MySQL 8.0 strictly rejects this syntax and throws a fatal error, killing the entire setup page.

Why It Happens

These ALTER TABLE statements exist to add columns to an existing table in case they are missing from an older install. On a fresh database created from scratch, the columns already exist from the CREATE TABLE statement that runs earlier in the script. The ALTER TABLE blocks are redundant for new installs — but MySQL 8.0 refuses to run them regardless.

The Fix

Open the setup script:

bashsudo nano /var/www/html/dvwa/includes/DBMS/MySQL.php

Find every block that contains ALTER TABLE ... IF NOT EXISTS and comment out the PHP code executing those statements. The blocks look like this:

php// Comment these out:
$db->query("ALTER TABLE users ADD COLUMN IF NOT EXISTS role VARCHAR(20) DEFAULT 'user'");
$db->query("ALTER TABLE users ADD COLUMN IF NOT EXISTS account_enabled TINYINT(1) DEFAULT 1");

Add // before each $db->query() line containing IF NOT EXISTS:

php// MySQL 8.0 compatibility fix — IF NOT EXISTS not supported in ALTER TABLE
// $db->query("ALTER TABLE users ADD COLUMN IF NOT EXISTS role VARCHAR(20) DEFAULT 'user'");
// $db->query("ALTER TABLE users ADD COLUMN IF NOT EXISTS account_enabled TINYINT(1) DEFAULT 1");

Save, return to setup.php, and click Create / Reset Database again. Setup completes successfully.

What This Does Not Break

Commenting out these lines skips the redundant column-addition step. Since the database was created fresh, all columns already exist from the CREATE TABLE statement. DVWA functions fully — all vulnerability modules work as expected.


Accessing DVWA

Navigate to: http://192.168.x.x/dvwa/login.php

Default credentials:


Username: admin
Password: password


After logging in, go to DVWA Security and set the security level:


Low — no input validation, easiest to exploit, start here
Medium — basic filtering applied
High — stronger controls
Impossible — secure implementation for comparison



Network Configuration Notes

The Ubuntu Target has dual network access:


VMnet1 (192.168.x.x) — isolated lab network, used for all lab communication
NAT — temporary internet access for package installation only, switched back to VMnet1 after each install


DVWA is only accessible from within the lab network. It is not exposed to the internet or the real home LAN.


Vulnerability Modules Available

ModuleWhat It TeachesSQL InjectionDatabase query manipulation, data extractionSQL Injection (Blind)Boolean and time-based blind injectionCross Site Scripting (Reflected)Client-side script injectionCross Site Scripting (Stored)Persistent XSS via database storageCSRFCross-site request forgery attacksFile InclusionLocal and remote file inclusionFile UploadMalicious file upload bypassCommand InjectionOS command execution via web inputBrute ForceCredential attacks against login formsInsecure CAPTCHACAPTCHA bypass techniques


Detection Integration

DVWA runs on the Ubuntu Target which has both a Wazuh agent and Suricata IDS active. Attack traffic against DVWA is visible in the Wazuh dashboard alongside host-based alerts — the same three-layer detection stack used throughout this lab:


Suricata — catches web attack signatures at the network level
Wazuh agent — logs Apache access and error events
auditd — captures system-level activity during exploitation



Next Steps


 Complete SQL injection module at all four security levels
 Complete XSS (reflected and stored) modules
 Set up Burp Suite on Kali and intercept DVWA traffic
 Practice IDOR and access control vulnerabilities
 Document each finding as a formal bug report in the format used on HackerOne
 Add Burp Suite setup guide to this repository



Related Lab Documentation


SOC Home Lab — Complete Build Report
Custom Wazuh Detection Rules
Network Forensics Analysis
Metasploit Incident Report — CVE-2011-2523

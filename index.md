---
title: AICLSA Linux Server Exam Report
layout: default
---

# Armour Infosec Certified Linux Server Administrator Exam Report
---
- **Rishabh Soni**
- **email@rishabhsoni.in**
- **09-June-2025**

## Table of Contents
---
- [Linux Server Administrator Exam Report](#linux-server-administrator-exam-report)
- [Question](#question)
- [Server Configurations](#server-configurations)
- [Client Configuration](#client-configuration)
- [Server Configuration Steps](#server-configuration-steps)
  - [Step 1: Initial Server Setup and DNS Configuration](#step-1-initial-server-setup-and-dns-configuration)
    - [1.1 Configure Network Settings](#11-configure-network-settings)
    - [1.2 Install and Configure DNS Server](#12-install-and-configure-dns-server)
  - [Step 2: LDAP Server Installation and Configuration](#step-2-ldap-server-installation-and-configuration)
    - [2.1 Install OpenLDAP Packages](#21-install-openldap-packages)
    - [2.2 Configure LDAP Database](#22-configure-ldap-database)
    - [2.3 Configure Access Control Lists](#23-configure-access-control-lists)
    - [2.4 Create LDAP Directory Structure](#24-create-ldap-directory-structure)
    - [2.5 Create Groups and Users](#25-create-groups-and-users)
  - [Step 3: FTP Server with LDAP Authentication](#step-3-ftp-server-with-ldap-authentication)
  - [Step 4: Centralized Home Directory with NFS](#step-4-centralized-home-directory-with-nfs)
  - [Step 5: Apache Web Server Installation](#step-5-apache-web-server-installation)
  - [Step 6: WordPress Installation and HTTPS Configuration](#step-6-wordpress-installation-and-https-configuration)
    - [6.1 Install PHP](#61-install-php)
    - [6.2 Install MySQL](#62-install-mysql)
    - [6.3 Install phpMyAdmin](#63-install-phpmyadmin)
    - [6.4 Install WordPress](#64-install-wordpress)
    - [6.5 Configure SSL and Virtual Hosts](#65-configure-ssl-and-virtual-hosts)
  - [Step 7: GitLab Installation with LDAP Integration](#step-7-gitlab-installation-with-ldap-integration)
  - [Step 8: LDAPS Implementation](#step-8-ldaps-implementation)
    - [Default VirtualHost Configuration](#default-virtualhost-configuration)
    - [8.1 Update Server-Side Services for LDAPS](#81-update-server-side-services-for-ldaps)
- [Client Configuration Steps](#client-configuration-steps)
  - [Step 1: Initial Client Setup](#step-1-initial-client-setup)
  - [Step 2: LDAP Client Configuration](#step-2-ldap-client-configuration)
    - [SSH Configuration for LDAP Authentication](#ssh-configuration-for-ldap-authentication)
    - [FTP Client Testing](#ftp-client-testing)
  - [Step 3: Centralized Home Directory Client Configuration](#step-3-centralized-home-directory-client-configuration)
  - [Step 4: LDAPS Client Configuration](#step-4-ldaps-client-configuration)
  - [Optional: Install GUI](#optional-install-gui)
- [Testing and Verification](#testing-and-verification)
  - [DNS Testing](#dns-testing)
  - [LDAP/LDAPS Authentication](#ldapldaps-authentication)
  - [Web Services Testing](#web-services-testing)
  - [Centralized Home Directory](#centralized-home-directory)
  - [Service Status Verification](#service-status-verification)

## Question
---
Your organization is focused on upgrading its Linux server infrastructure, prioritizing both security and operational efficiency. Begin by setting up an LDAP server to centralize authentication across the network. Then, deploy WordPress and ensure it is securely accessible over HTTPS on port 443 using a designated domain name. Additionally, integrate GitLab with LDAP for user authentication and make it available via its own domain. To finalize the setup, implement a file-sharing service for user home directories using FTP with LDAP-based authentication, and configure the SSH server to authenticate users through LDAP as well.

## Server Configurations
---
**OS:** CentOS 9  
**IPv4:** 192.168.1.128/24  
**Gateway:** 192.168.1.1  
**Hostname:** centos.rs.local  
**Domains:** rs.local, ldap.rs.local, wordpress.rs.local, gitlab.rs.local  

### Credentials
---
**Server Users:**

| Users | Password   | Type  |
| ----- | ---------- | ----- |
| root  | Rish@bh123 | admin |
| rsoni | Rish@bh123 | admin |


**LDAP Admin:**

| User                    | Password   |
| ----------------------- | ---------- |
| cn=admin,dc=rs,dc=local | Rish@bh123 |


**LDAP Users:**

| Department | Username | Password   | UID  | GID  | Group Name | Home Directory |
| ---------- | -------- | ---------- | ---- | ---- | ---------- | -------------- |
| HR         | hr1      | Hr1@123    | 3001 | 2005 | hr         | /home/hr1      |
| HR         | hr2      | Hr2@123    | 3002 | 2005 | hr         | /home/hr2      |
| IT         | it1      | It1@123    | 3003 | 2004 | it         | /home/it1      |
| IT         | it2      | It2@123    | 3004 | 2004 | it         | /home/it2      |
| Admin      | admin1   | Admin1@123 | 3005 | 2003 | admin      | /home/admin1   |
| Admin      | admin2   | Admin2@123 | 3006 | 2003 | admin      | /home/admin2   |


**LDAP Groups**

| Group Name | GID  | Members        | Purpose              |
| ---------- | ---- | -------------- | -------------------- |
| admin      | 2003 | admin1, admin2 | Administrative users |
| it         | 2004 | it1, it2       | IT department users  |
| hr         | 2005 | hr1, hr2       | HR department users  |


**Service Credentials**

| Service    | Username     | Password   | Access URL/Port            | Notes                   |
| ---------- | ------------ | ---------- | -------------------------- | ----------------------- |
| MySQL      | root         | Rish@bh123 | localhost:3306             | Database root access    |
| WordPress  | wpadmin      | Rish@bh123 | https://wordpress.rs.local | WordPress Administrator |
| GitLab     | root         | Rish@bh123 | https://gitlab.rs.local    | GitLab Administrator    |
| phpMyAdmin | root (MySQL) | Rish@bh123 | Non accessible by URL      | Web database management |


**Service Access Matrix**

| Service       | Local Users    | LDAP Users       | Authentication Method | Access Type     |
| ------------- | -------------- | ---------------- | --------------------- | --------------- |
| SSH           | ✓ root, rsoni  | ✓ All LDAP users | PAM + SSSD            | Terminal access |
| FTP           | ✗              | ✓ All LDAP users | PAM + SSSD            | File transfer   |
| GitLab        | ✓ root (admin) | ✓ All LDAP users | LDAP/LDAPS            | Web access      |
| WordPress     | ✓ wpadmin      | ✗                | Local WordPress DB    | Web access      |
| NFS Home Dirs | N/A            | ✓ All LDAP users | LDAP UID/GID          | Auto-mounted    |


**WordPress Installation Details**

| Configuration Item | Value                      |
| ------------------ | -------------------------- |
| Access URL         | https://wordpress.rs.local |
| Database Name      | wordpress.rs.local         |
| Database Username  | root                       |
| Database Password  | Rish@bh123                 |
| Table Prefix       | wp_rs_                     |
| Site Title         | wordpress.rs.local         |
| Admin Username     | wpadmin                    |
| Admin Password     | Rish@bh123                 |

---
## Client Configuration
---
**OS:** CentOS 9  
**IPv4:** 192.168.1.201/24  
**Gateway:** 192.168.1.1  
**DNS:** 192.168.1.128  
**Hostname:** c1.rs.local  

### Credential

| Users        | Password         |
| ------------ | ---------------- |
| root         | Rish@bh123       |
| rsoni        | Rish@bh123       |
| *LDAP users* | *Same as server* |

---
---
## Server Configuration Steps
---
### Step 1: Initial Server Setup and DNS Configuration
---
#### 1.1 Configure Network Settings
---
Set static IP using nmtui

```bash
nmtui
```

Configure:
- IPv4: 192.168.1.128/24
- Gateway: 192.168.1.1
- DNS: 127.0.0.1

Set hostname

```bash
hostnamectl set-hostname centos.rs.local
```

```bash
hostnamectl status
```

Enable root SSH access

```bash
vim /etc/ssh/sshd_config
```

Set: PermitRootLogin yes

```bash
systemctl restart sshd
```

Install basic tools

```bash
dnf install bash* wget unzip net-tools
```

#### 1.2 Install and Configure DNS Server
---
Install BIND

```bash
dnf install bind bind-utils
```

Configure named.conf

```bash
vim /etc/named.conf
```

Add the following configuration:
```
options {
    listen-on port 53 { 127.0.0.1; 192.168.1.128; };
    allow-query { localhost; 192.168.1.0/24; };
    forwarders { 8.8.8.8; 8.8.4.4; };
};
```

Configure zones

```bash
vim /etc/named.rfc1912.zones
```

Add:
```
zone "rs.local" {
    type master;
    file "/var/named/rs.local.zone";
};

zone "1.168.192.in-addr.arpa" {
    type master;
    file "/var/named/1.168.192.rev";
};
```

Create forward zone file

```bash
vim /var/named/rs.local.zone
```

```
$TTL 86400
@   IN  SOA centos.rs.local. admin.rs.local. (
        2024010103  ; Serial
        3600        ; Refresh
        1800        ; Retry
        604800      ; Expire
        86400       ; Minimum TTL
)
    IN  NS  centos.rs.local.
    IN  A   192.168.1.128

centos      IN  A       192.168.1.128
ldap        IN  CNAME   centos
wordpress   IN  A       192.168.1.128
www.wordpress IN CNAME  wordpress
gitlab      IN  A       192.168.1.128
www.gitlab  IN CNAME    gitlab
c1          IN  A       192.168.1.201
```

Create reverse zone file

```bash
vim /var/named/1.168.192.rev
```

```
$TTL 86400
@   IN  SOA centos.rs.local. admin.rs.local. (
        2024010101  ; Serial
        3600        ; Refresh
        1800        ; Retry
        604800      ; Expire
        86400       ; Minimum TTL
)
    IN  NS  centos.rs.local.

128  IN  PTR centos.rs.local.
201  IN  PTR c1.rs.local.
```

Enable and start DNS service

```bash
systemctl enable named --now
```

```bash
firewall-cmd --add-service=dns --permanent
```

```bash
firewall-cmd --reload
```

Test DNS

```bash
dig centos.rs.local
```

```bash
dig ldap.rs.local
```

```bash
dig wordpress.rs.local
```

```bash
dig gitlab.rs.local
```

### Step 2: LDAP Server Installation and Configuration
---
#### 2.1 Install OpenLDAP Packages
---
Install required repositories

```bash
dnf install -y epel-release
```

```bash
dnf config-manager --set-enabled crb
```

```bash
dnf makecache
```

Install OpenLDAP

```bash
dnf install -y openldap-servers openldap-clients openldap-devel sssd-tools
```

```bash
systemctl enable slapd --now
```

Configure firewall

```bash
firewall-cmd --add-service=ldap --permanent
```

```bash
firewall-cmd --add-service=ldaps --permanent
```

```bash
firewall-cmd --reload
```

#### 2.2 Configure LDAP Database
---
Create LDIF directory

```bash
mkdir /root/ldif
```

Generate admin password

```bash
slappasswd
```

Enter password: Rish@bh123
Output: {SSHA}OUMtfIbsWAltpieoq8WuXMooVmwM8nW2

Configure database

```bash
vim /root/ldif/db-config.ldif
```

```ldif
dn: olcDatabase={2}mdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=rs,dc=local

dn: olcDatabase={2}mdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=admin,dc=rs,dc=local

dn: olcDatabase={2}mdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}OUMtfIbsWAltpieoq8WuXMooVmwM8nW2
```

Apply configuration

```bash
ldapmodify -Y EXTERNAL -H ldapi:/// -f /root/ldif/db-config.ldif
```

Load schemas

```bash
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
```

```bash
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
```

```bash
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
```

#### 2.3 Configure Access Control Lists
---
```bash
vim /root/ldif/acl.ldif
```

```ldif
dn: olcDatabase={2}mdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to attrs=userPassword
  by self write
  by anonymous auth
  by dn="cn=admin,dc=rs,dc=local" write
  by * none
olcAccess: {1}to attrs=shadowLastChange
  by self write
  by * read
olcAccess: {2}to *
  by dn="cn=admin,dc=rs,dc=local" write
  by self read
  by * read
```

```bash
ldapmodify -Y EXTERNAL -H ldapi:/// -f /root/ldif/acl.ldif
```

Check
```bash
ldapsearch -Y EXTERNAL -H ldapi:/// -b "olcDatabase={2}mdb,cn=config" olcAccess
```
#### 2.4 Create LDAP Directory Structure
---
Create base domain

```bash
vim /root/ldif/base.ldif
```

```ldif
dn: dc=rs,dc=local
objectClass: top
objectClass: dcObject
objectClass: organization
o: RS
dc: rs
```

```bash
ldapadd -x -D "cn=admin,dc=rs,dc=local" -W -f /root/ldif/base.ldif
```

Enter password: Rish@bh123

Create organizational units

```bash
vim /root/ldif/base_structure.ldif
```

```ldif
dn: ou=it,dc=rs,dc=local
objectClass: organizationalUnit
ou: it

dn: ou=hr,dc=rs,dc=local
objectClass: organizationalUnit
ou: hr

dn: ou=admin,dc=rs,dc=local
objectClass: organizationalUnit
ou: admin

dn: ou=groups,dc=rs,dc=local
objectClass: organizationalUnit
ou: groups
```

```bash
ldapadd -x -D "cn=admin,dc=rs,dc=local" -W -f /root/ldif/base_structure.ldif
```

#### 2.5 Create Groups and Users
---
Create groups

```bash
vim /root/ldif/create_group.ldif
```

```ldif
dn: cn=it,ou=groups,dc=rs,dc=local
objectClass: posixGroup
cn: it
gidNumber: 2004

dn: cn=hr,ou=groups,dc=rs,dc=local
objectClass: posixGroup
cn: hr
gidNumber: 2005

dn: cn=admin,ou=groups,dc=rs,dc=local
objectClass: posixGroup
cn: admin
gidNumber: 2003
```

```bash
ldapadd -x -D "cn=admin,dc=rs,dc=local" -W -f /root/ldif/create_group.ldif
```

Generate user passwords

```bash
slappasswd -s 'Hr1@123'
```
{SSHA}us8Pn4pKI+iDSGFyyHcsaoLig5WZTBTl

```bash
slappasswd -s 'Hr2@123'
```
{SSHA}S9/u4SZLPNVpLNVe+7DZ/GQxL48rOm2W

```bash
slappasswd -s 'It1@123'
```
{SSHA}T31pWHMqcK7Av6O7wMk2UqbpzliYIKYg

```bash
slappasswd -s 'It2@123'
```
{SSHA}YuhinO4tIYnru9P3OeUQuC99R2XHdiW5

```bash
slappasswd -s 'Admin1@123'
```
{SSHA}NrNSslavho991v3whXrr8lyiyU4JD89T

```bash
slappasswd -s 'Admin2@123'
```
{SSHA}L4C1pXV4+hvAHk+JyRi+Kuk9eAAlSdok

Create users

```bash
vim /root/ldif/add_users.ldif
```

```ldif
dn: uid=hr1,ou=hr,dc=rs,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
cn: HR One
sn: One
uid: hr1
uidNumber: 3001
gidNumber: 2005
homeDirectory: /home/hr1
loginShell: /bin/bash
userPassword: {SSHA}us8Pn4pKI+iDSGFyyHcsaoLig5WZTBTl

dn: uid=hr2,ou=hr,dc=rs,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
cn: HR Two
sn: Two
uid: hr2
uidNumber: 3002
gidNumber: 2005
homeDirectory: /home/hr2
loginShell: /bin/bash
userPassword: {SSHA}S9/u4SZLPNVpLNVe+7DZ/GQxL48rOm2W

dn: uid=it1,ou=it,dc=rs,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
cn: IT One
sn: One
uid: it1
uidNumber: 3003
gidNumber: 2004
homeDirectory: /home/it1
loginShell: /bin/bash
userPassword: {SSHA}T31pWHMqcK7Av6O7wMk2UqbpzliYIKYg

dn: uid=it2,ou=it,dc=rs,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
cn: IT Two
sn: Two
uid: it2
uidNumber: 3004
gidNumber: 2004
homeDirectory: /home/it2
loginShell: /bin/bash
userPassword: {SSHA}YuhinO4tIYnru9P3OeUQuC99R2XHdiW5

dn: uid=admin1,ou=admin,dc=rs,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
cn: Admin One
sn: One
uid: admin1
uidNumber: 3005
gidNumber: 2003
homeDirectory: /home/admin1
loginShell: /bin/bash
userPassword: {SSHA}NrNSslavho991v3whXrr8lyiyU4JD89T

dn: uid=admin2,ou=admin,dc=rs,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
cn: Admin Two
sn: Two
uid: admin2
uidNumber: 3006
gidNumber: 2003
homeDirectory: /home/admin2
loginShell: /bin/bash
userPassword: {SSHA}L4C1pXV4+hvAHk+JyRi+Kuk9eAAlSdok
```

```bash
ldapadd -x -D "cn=admin,dc=rs,dc=local" -W -f /root/ldif/add_users.ldif
```

Add users to groups

```bash
vim /root/ldif/add_member_to_group.ldif
```

```ldif
dn: cn=it,ou=groups,dc=rs,dc=local
changetype: modify
add: memberUid
memberUid: it1
memberUid: it2

dn: cn=hr,ou=groups,dc=rs,dc=local
changetype: modify
add: memberUid
memberUid: hr1
memberUid: hr2

dn: cn=admin,ou=groups,dc=rs,dc=local
changetype: modify
add: memberUid
memberUid: admin1
memberUid: admin2
```

```bash
ldapmodify -x -D "cn=admin,dc=rs,dc=local" -W -f /root/ldif/add_member_to_group.ldif
```

Test LDAP authentication

```bash
ldapwhoami -x -D "uid=it1,ou=it,dc=rs,dc=local" -W
```

Enter password: It1@123

LDAP Directory Structure Tree:
```
dc=rs,dc=local
├── ou=hr
│   ├── uid=hr1 (member of hr group)
│   └── uid=hr2 (member of hr group)
├── ou=it
│   ├── uid=it1 (member of it group)
│   └── uid=it2 (member of it group)
├── ou=admin
│   ├── uid=admin1 (member of admin group)
│   └── uid=admin2 (member of admin group)
└── ou=groups
    ├── cn=hr      (memberUid: hr1, hr2)
    ├── cn=it      (memberUid: it1, it2)
    └── cn=admin   (memberUid: admin1, admin2)
```
### Step 3: FTP Server with LDAP Authentication
---
Install vsftpd and SSSD

```bash
dnf install vsftpd ftp sssd sssd-ldap -y
```

Configure SSSD

```bash
vim /etc/sssd/sssd.conf
```

```ini
[sssd]
config_file_version = 2
services = nss, pam
domains = rs.local

[domain/rs.local]
id_provider = ldap
auth_provider = ldap
ldap_uri = ldap://localhost
ldap_search_base = dc=rs,dc=local
ldap_schema = rfc2307
ldap_tls_reqcert = never
ldap_id_use_start_tls = false
ldap_auth_disable_tls_never_use_in_production = true
cache_credentials = true
enumerate = true
ldap_default_bind_dn = cn=admin,dc=rs,dc=local
ldap_default_authtok = Rish@bh123

[nss]
homedir_substring = /home
filter_groups = root
filter_users = root

[pam]
```

Set permissions and enable SSSD

```bash
chmod 600 /etc/sssd/sssd.conf
```

```bash
systemctl enable sssd --now
```

```bash
authselect select sssd with-mkhomedir --force
```

Configure vsftpd

```bash
vim /etc/vsftpd/vsftpd.conf
```

```
anonymous_enable=NO
local_enable=YES
write_enable=YES
chroot_local_user=YES
allow_writeable_chroot=YES
use_localtime=YES
xferlog_enable=YES
xferlog_std_format=YES
pam_service_name=vsftpd
ssl_enable=NO
listen=YES
listen_ipv6=NO
userlist_enable=YES
userlist_deny=NO
userlist_file=/etc/vsftpd/user_list_rs
tcp_wrappers=NO
local_root=/home/$USER
user_sub_token=$USER
```

Create FTP user access list

```bash
vim /etc/vsftpd/user_list_rs
```

```
it1
it2
admin1
admin2
hr1
hr2
```

Create home directories

```bash
mkdir -p /home/{it1,it2,hr1,hr2,admin1,admin2}
```

Enable services

```bash
systemctl enable vsftpd --now
```

```bash
systemctl enable oddjobd --now
```

Configure firewall and SELinux

```bash
firewall-cmd --add-service=ftp --permanent
```

```bash
firewall-cmd --reload
```

```bash
getsebool -a | grep ftp
```

```bash
setsebool -P ftpd_full_access on
```

### Step 4: Centralized Home Directory with NFS
---
Install NFS

```bash
dnf install nfs-utils -y
```

Configure NFS exports

```bash
vim /etc/exports
```

```
/home *(rw,sync,no_root_squash,no_subtree_check)
```

Enable NFS services

```bash
systemctl enable nfs-server --now
```

```bash
systemctl enable rpcbind --now
```

```bash
exportfs -arv
```

Configure firewall

```bash
firewall-cmd --add-service=nfs --permanent
```

```bash
firewall-cmd --add-service=rpc-bind --permanent
```

```bash
firewall-cmd --add-service=mountd --permanent
```

```bash
firewall-cmd --reload
```

### Step 5: Apache Web Server Installation
---
Install Apache and SSL module

```bash
dnf install httpd mod_ssl
```

```bash
systemctl enable httpd
```

```bash
systemctl start httpd
```

Disable welcome page

```bash
mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf.notinclude
```

Configure firewall

```bash
firewall-cmd --add-service=http --permanent
```

```bash
firewall-cmd --add-service=https --permanent
```

```bash
firewall-cmd --reload
```

### Step 6: WordPress Installation and HTTPS Configuration
---
#### 6.1 Install PHP
---
Install PHP repositories

```bash
dnf install epel-release yum-utils
```

```bash
dnf install http://rpms.remirepo.net/enterprise/remi-release-9.rpm
```

Install PHP and modules

```bash
dnf install php php-common php-cli php-opcache php-gd php-curl php-mysqlnd php-xml php-mbstring php-pear php-pecl-http php-session
```

```bash
systemctl restart httpd
```

#### 6.2 Install MySQL
---
Download MySQL repository

```bash
wget https://dev.mysql.com/get/mysql84-community-release-el9-1.noarch.rpm
```

```bash
dnf install ./mysql84-community-release-el9-1.noarch.rpm
```

Enable MySQL 8.0 repository

```bash
dnf config-manager --enable mysql80-community
```

Install MySQL

```bash
yum install mysql-community-server mysql-community-devel
```

Enable and start MySQL

```bash
systemctl enable mysqld.service
```

```bash
systemctl restart mysqld.service
```

Get temporary root password

```bash
grep 'temporary password' /var/log/mysqld.log
```

Note the temporary password (e.g., KwWyoOPqn3?i)

Secure MySQL installation

```bash
mysql_secure_installation
```

Change root password to: Rish@bh123
Remove anonymous users: y
Disallow root login remotely: y
Remove test database: y
Reload privilege tables: y

#### 6.3 Install phpMyAdmin
---
Download and install phpMyAdmin

```bash
wget https://files.phpmyadmin.net/phpMyAdmin/5.1.0/phpMyAdmin-5.1.0-all-languages.zip
```

```bash
unzip phpMyAdmin-5.1.0-all-languages.zip
```

```bash
mv phpMyAdmin-5.1.0-all-languages /var/www/html/phpmyadmin
```

Configure phpMyAdmin

```bash
cp /var/www/html/phpmyadmin/config.sample.inc.php /var/www/html/phpmyadmin/config.inc.php
```

Generate blowfish secret

```bash
dnf install pwgen
```

```bash
pwgen 32 -1
```

Use generated password (e.g., OoFahguchoh7ohsohThaelau0Olah8ch)

Edit configuration

```bash
vim /var/www/html/phpmyadmin/config.inc.php
```

```php
####line 16 
-$cfg['blowfish_secret'] = 'eephoo8ey8EuQu6Jiewee1ietaew6Eit'; /* YOU MUST FILL IN THIS FOR COOKIE AUTH! */
```

Update line 16 with generated blowfish_secret

Set permissions

```bash
chown -Rv apache:apache /var/www/html/phpmyadmin
```

```bash
systemctl restart httpd
```

Create WordPress database via phpMyAdmin
Access: http://192.168.1.128/phpmyadmin/
Create database: wordpress.rs.local

#### 6.4 Install WordPress
---
Download and install WordPress

```bash
wget https://wordpress.org/latest.zip
```

```bash
unzip latest.zip
```

```bash
mv wordpress/ /var/www/html/
```

Set permissions

```bash
chown -Rv apache:apache /var/www/html/wordpress/
```

```bash
chmod -Rv 0755 /var/www/html/wordpress/wp-includes/ /var/www/html/wordpress/wp-admin/js/ /var/www/html/wordpress/wp-content/themes/ /var/www/html/wordpress/wp-content/plugins/
```

```bash
systemctl restart httpd
```

#### 6.5 Configure SSL and Virtual Hosts
---
Create SSL directory

```bash
mkdir /opt/ssl
```

```bash
cd /opt/ssl/
```

Generate self-signed certificate

```bash
openssl req -x509 -newkey rsa:2048 -keyout rs.local.key.pem -out rs.local.cert.pem -days 365
```

Passphrase: Rish@bh123
Fill in certificate details with appropriate values

Copy certificates to proper locations

```bash
cp -v /opt/ssl/rs.local.cert.pem /etc/pki/tls/certs/rs.local.cert.pem
```

```bash
cp -v /opt/ssl/rs.local.key.pem /etc/pki/tls/private/rs.local.key.pem
```

Configure Apache main configuration

```bash
vim /etc/httpd/conf/httpd.conf
```

Change: Listen 192.168.1.128:80
In <Directory "/var/www/html"> section:
    Options -Indexes +FollowSymLinks
    AllowOverride All

Create WordPress virtual host

```bash
vim /etc/httpd/conf.d/wordpress.rs.conf
```

```apache
<VirtualHost 192.168.1.128:80>
    ServerName wordpress.rs.local
    ServerAlias www.wordpress.rs.local

    <If "%{HTTP_HOST} != 'wordpress.rs.local' && %{HTTP_HOST} != 'www.wordpress.rs.local'">
        Require all denied
    </If>

    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php index.html
    Redirect permanent / https://wordpress.rs.local/

    <Directory /var/www/html/wordpress/>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>

<VirtualHost 192.168.1.128:443>
    ServerName wordpress.rs.local
    ServerAlias www.wordpress.rs.local

    <If "%{HTTP_HOST} != 'wordpress.rs.local' && %{HTTP_HOST} != 'www.wordpress.rs.local'">
        Require all denied
    </If>

    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php index.html

    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/rs.local.cert.pem
    SSLCertificateKeyFile /etc/pki/tls/private/rs.local.key.pem

    <Directory /var/www/html/wordpress/>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog /var/log/httpd/wordpress-ssl-error.log
    CustomLog /var/log/httpd/wordpress-ssl-access.log combined
</VirtualHost>
```

Restart Apache

```bash
systemctl restart httpd
```

Complete WordPress installation via web interface
Access: https://wordpress.rs.local
Database Name: wordpress.rs.local
Username: root
Password: Rish@bh123
Table Prefix: wp_rs_
Site Title: wordpress.rs.local
Admin Username: wpadmin
Admin Password: Rish@bh123

### Step 7: GitLab Installation with LDAP Integration
---
Install required packages

```bash
dnf install -y curl policycoreutils perl
```

Download and install GitLab repository

```bash
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh -o gitlab-repo-installation-script.sh
```

```bash
chmod +x gitlab-repo-installation-script.sh
```

```bash
./gitlab-repo-installation-script.sh
```

Install GitLab

```bash
dnf install -y gitlab-ce
```

Configure GitLab

```bash
vim /etc/gitlab/gitlab.rb
```

```ruby
# Disable GitLab's built-in nginx
nginx['enable'] = false

# Configure GitLab to work with Apache
gitlab_workhorse['listen_network'] = "tcp"
gitlab_workhorse['listen_addr'] = "127.0.0.1:8181"

# Set external URL
external_url 'https://gitlab.rs.local'

# LDAP Configuration
gitlab_rails['ldap_enabled'] = true
gitlab_rails['prevent_ldap_sign_in'] = false

gitlab_rails['ldap_servers'] = YAML.load <<-'EOS'
  main:
    label: 'LDAP'
    host: 'centos.rs.local'
    port: 389
    uid: 'uid'
    bind_dn: 'cn=admin,dc=rs,dc=local'
    password: 'Rish@bh123'
    encryption: 'plain'
    verify_certificates: false
    smartcard_auth: false
    active_directory: false
    allow_username_or_email_login: false
    lowercase_usernames: false
    block_auto_created_users: false
    base: 'dc=rs,dc=local'
    user_filter: ''
    attributes:
      username: ['uid', 'userid', 'sAMAccountName']
      email: ['mail', 'email', 'userPrincipalName']
      name: 'cn'
      first_name: 'givenName'
      last_name: 'sn'
    group_base: 'ou=groups,dc=rs,dc=local'
    admin_group: 'admin'
    sync_ssh_keys: false
EOS

gitlab_rails['smtp_enable'] = false
```

Create GitLab virtual host

```bash
vim /etc/httpd/conf.d/gitlab.rs.conf
```

```apache
<VirtualHost 192.168.1.128:80>
    ServerName gitlab.rs.local
    ServerAlias www.gitlab.rs.local

    <If "%{HTTP_HOST} != 'gitlab.rs.local' && %{HTTP_HOST} != 'www.gitlab.rs.local'">
        Require all denied
    </If>

    Redirect permanent / https://gitlab.rs.local/
</VirtualHost>

<VirtualHost 192.168.1.128:443>
    ServerName gitlab.rs.local
    ServerAlias www.gitlab.rs.local

    <If "%{HTTP_HOST} != 'gitlab.rs.local' && %{HTTP_HOST} != 'www.gitlab.rs.local'">
        Require all denied
    </If>

    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/rs.local.cert.pem
    SSLCertificateKeyFile /etc/pki/tls/private/rs.local.key.pem

    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8181/
    ProxyPassReverse / http://127.0.0.1:8181/

    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader set X-Forwarded-Ssl on

    ErrorLog /var/log/httpd/gitlab-ssl-error.log
    CustomLog /var/log/httpd/gitlab-ssl-access.log combined
</VirtualHost>
```

Reconfigure and start GitLab

```bash
gitlab-ctl reconfigure
```

```bash
gitlab-ctl start
```

Get initial root password

```bash
cat /etc/gitlab/initial_root_password
```

Save the generated password

Test LDAP connection

```bash
gitlab-rake gitlab:ldap:check
```

Access GitLab and change root password
URL: https://gitlab.rs.local
Username: root
Password: (from initial_root_password file)
Change password to: Rish@bh123

### Step 8: LDAPS Implementation
---
Create SSL certificates for LDAP

```bash
mkdir -p /etc/openldap/certs
```

```bash
cd /etc/openldap/certs
```

Generate certificate

```bash
openssl req -new -x509 -nodes -out ldapserver.crt -keyout ldapserver.key -days 365 -subj "/C=IN/ST=MP/L=City/O=RS/CN=centos.rs.local"
```

Set permissions

```bash
chown ldap:ldap /etc/openldap/certs/*
```

```bash
chmod 600 /etc/openldap/certs/ldapserver.key
```

```bash
chmod 644 /etc/openldap/certs/ldapserver.crt
```

Copy certificate to trust store

```bash
cp /etc/openldap/certs/ldapserver.crt /etc/pki/ca-trust/source/anchors/
```

```bash
update-ca-trust
```

Configure LDAP for TLS

```bash
vim ~/ldif/tls-config.ldif
```

```ldif
dn: cn=config
changetype: modify
add: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/openldap/certs/ldapserver.crt
-
add: olcTLSCertificateFile
olcTLSCertificateFile: /etc/openldap/certs/ldapserver.crt
-
add: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/openldap/certs/ldapserver.key
```

Apply TLS configuration

```bash
ldapmodify -Y EXTERNAL -H ldapi:/// -f ~/ldif/tls-config.ldif
```

Configure LDAP to listen on LDAPS port

```bash
vim /etc/sysconfig/slapd
```

```
SLAPD_URLS="ldapi:/// ldap:/// ldaps:///"
```

Remove ldap:/// to only allow secure connections:

```bash
SLAPD_URLS="ldapi:/// ldaps:///"
```

Restart LDAP service

```bash
systemctl restart slapd
```

Verify LDAPS port

```bash
ss -nltp | grep 636
```

Test LDAPS connection

```bash
ldapsearch -x -H ldaps://centos.rs.local -b "dc=rs,dc=local" -D "cn=admin,dc=rs,dc=local" -W
```


### Default VirtualHost Configuration
---

This configuration creates a catch-all VirtualHost that denies access to any hostname not explicitly configured, preventing unauthorized access via IP address or unintended hostnames.

```bash
vim /etc/httpd/conf.d/00-default.conf
```

```apache
<VirtualHost 192.168.1.128:80>
    ServerName default
    DocumentRoot /var/www/default

    # Deny all requests
    <Directory /var/www/default>
        Require all denied
    </Directory>

    # Alternative: Return 403 immediately for any request
    <Location />
        Require all denied
    </Location>

    ErrorLog /var/log/httpd/default-error.log
    CustomLog /var/log/httpd/default-access.log combined
</VirtualHost>

<VirtualHost 192.168.1.128:443>
    ServerName default
    DocumentRoot /var/www/default

    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/rs.local.cert.pem
    SSLCertificateKeyFile /etc/pki/tls/private/rs.local.key.pem

    # Deny all requests
    <Directory /var/www/default>
        Require all denied
    </Directory>

    # Alternative: Return 403 immediately for any request
    <Location />
        Require all denied
    </Location>

    ErrorLog /var/log/httpd/default-ssl-error.log
    CustomLog /var/log/httpd/default-ssl-access.log combined
</VirtualHost>
```

#### 8.1 Update Server-Side Services for LDAPS
---
Update SSSD configuration

```bash
vim /etc/sssd/sssd.conf
```

```ini
[sssd]
config_file_version = 2
services = nss, pam
domains = rs.local

[domain/rs.local]
id_provider = ldap
auth_provider = ldap
ldap_uri = ldaps://centos.rs.local:636
ldap_search_base = dc=rs,dc=local
ldap_schema = rfc2307
ldap_tls_reqcert = allow
ldap_tls_cacert = /etc/openldap/certs/ldapserver.crt
ldap_id_use_start_tls = false
cache_credentials = true
enumerate = true
ldap_default_bind_dn = cn=admin,dc=rs,dc=local
ldap_default_authtok = Rish@bh123

[nss]
homedir_substring = /home
filter_groups = root
filter_users = root

[pam]
```

Restart SSSD

```bash
systemctl restart sssd
```

```bash
sssctl cache-expire -E
```

Update GitLab for LDAPS

```bash
vim /etc/gitlab/gitlab.rb
```

Update the LDAP configuration section:
```ruby
gitlab_rails['ldap_servers'] = YAML.load <<-'EOS'
  main:
    label: 'LDAP'
    host: 'centos.rs.local'
    port: 636                                    # Changed from 389
    uid: 'uid'
    bind_dn: 'cn=admin,dc=rs,dc=local'
    password: 'Rish@bh123'
    encryption: 'simple_tls'                     # Changed from 'plain'
    verify_certificates: true                    # Changed from false
    ca_file: '/etc/openldap/certs/ldapserver.crt'  # Added
    ssl_version: 'TLSv1_2'                       # Added
    smartcard_auth: false
    active_directory: false
    allow_username_or_email_login: false
    lowercase_usernames: false
    block_auto_created_users: false
    base: 'dc=rs,dc=local'
    user_filter: ''
    attributes:
      username: ['uid', 'userid', 'sAMAccountName']
      email: ['mail', 'email', 'userPrincipalName']
      name: 'cn'
      first_name: 'givenName'
      last_name: 'sn'
    group_base: 'ou=groups,dc=rs,dc=local'
    admin_group: 'admin'
    sync_ssh_keys: false
EOS
```

Copy certificate for GitLab

```bash
cp /etc/openldap/certs/ldapserver.crt /etc/gitlab/trusted-certs/
```

Reconfigure GitLab

```bash
gitlab-ctl reconfigure
```

```bash
gitlab-ctl restart
```

Test GitLab LDAP connection

```bash
gitlab-rake gitlab:ldap:check
```

Fix Apache startup issue

```bash
mkdir -p /etc/systemd/system/httpd.service.d/
```

```bash
vim /etc/systemd/system/httpd.service.d/network-wait.conf
```

```ini
[Unit]
After=network-online.target
Wants=network-online.target
```

Reload systemd and restart Apache

```bash
systemctl daemon-reload
```

```bash
systemctl restart httpd
```

---

---
---
# Client Configuration Steps
---
### Step 1: Initial Client Setup
---
Configure network settings

```bash
nmtui
```

IPv4: 192.168.1.201/24
Gateway: 192.168.1.1
DNS: 192.168.1.128

Set hostname

```bash
hostnamectl set-hostname c1.rs.local
```

Install basic tools

```bash
dnf install bash* wget unzip net-tools bind-utils vim
```

Test DNS resolution

```bash
nslookup centos.rs.local
```

```bash
ping centos.rs.local
```

### Step 2: LDAP Client Configuration
---
Install LDAP client packages

```bash
dnf install openldap-clients sssd sssd-ldap oddjob-mkhomedir sssd-tools -y
```

Configure SSSD

```bash
vim /etc/sssd/sssd.conf
```

```ini
[sssd]
config_file_version = 2
services = nss, pam, sudo
domains = rs.local

[domain/rs.local]
debug_level = 9
id_provider = ldap
auth_provider = ldap
ldap_uri = ldap://centos.rs.local:389
ldap_search_base = dc=rs,dc=local
ldap_schema = rfc2307
ldap_tls_reqcert = never
ldap_auth_disable_tls_never_use_in_production = true
ldap_id_use_start_tls = false
cache_credentials = true
enumerate = true
ldap_default_bind_dn = cn=admin,dc=rs,dc=local
ldap_default_authtok = Rish@bh123

[nss]
homedir_substring = /home
filter_groups = root
filter_users = root

[pam]
offline_credentials_expiration = 60

[sudo]
```

Set permissions

```bash
chmod 600 /etc/sssd/sssd.conf
```

```bash
chown root:root /etc/sssd/sssd.conf
```

Enable SSSD

```bash
systemctl enable sssd --now
```

Configure authentication
- Configure pam to use sssd for authentication and auto home dir creation on user login
```bash
authselect select sssd with-mkhomedir --force
```

```bash
systemctl enable oddjobd --now
```

Test LDAP user lookup

```bash
getent passwd it1
```

Test FTP access

```bash
ftp centos.rs.local
```

Login with LDAP credentials

#### SSH Configuration for LDAP Authentication
---
Verify SSH is configured to use PAM for LDAP authentication

```bash
vim /etc/ssh/sshd_config
```

Ensure the following is set:
```
UsePAM yes
```

Restart SSH service to apply changes

```bash
systemctl restart sshd
```

#### FTP Client Testing
---
Install FTP client

```bash
dnf install ftp
```

Test FTP connection using hostname

```bash
ftp centos.rs.local
```

Alternative test using IP address

```bash
ftp 192.168.1.128
```

Test FTP commands after login:
- Username: hr1
- Password: Hr1@123

```bash
pwd
```

```bash
ls
```

Create a test file and upload

```bash
put testfile.txt
```

Download a file from server

```bash
get testfile.txt
```

Exit FTP session

```bash
quit
```


### Step 3: Centralized Home Directory Client Configuration
---
Install NFS and AutoFS

```bash
dnf install nfs-utils autofs -y
```

Remove local home directories

```bash
rm -rf /home/it1 /home/it2 /home/hr1 /home/hr2 /home/admin1 /home/admin2
```

Configure AutoFS master

```bash
vim /etc/auto.master
```

Add:
```
/home   /etc/auto.home  --timeout=60
```

Configure AutoFS home mapping

```bash
vim /etc/auto.home
```

```
* -rw,soft,intr centos.rs.local:/home/&
```

Enable AutoFS

```bash
systemctl enable autofs --now
```

Test centralized home directories

```bash
ls /home/hr1
```

```bash
su - it1
```

### Step 4: LDAPS Client Configuration
---
Copy LDAP certificate from server

```bash
scp root@centos.rs.local:/etc/openldap/certs/ldapserver.crt /tmp/
```

```bash
cp /tmp/ldapserver.crt /etc/pki/ca-trust/source/anchors/
```

```bash
update-ca-trust
```

Update SSSD for LDAPS

```bash
vim /etc/sssd/sssd.conf
```

```ini
[sssd]
config_file_version = 2
services = nss, pam, sudo
domains = rs.local

[domain/rs.local]
debug_level = 9
id_provider = ldap
auth_provider = ldap
ldap_uri = ldaps://centos.rs.local:636
ldap_search_base = dc=rs,dc=local
ldap_schema = rfc2307
ldap_tls_reqcert = allow
ldap_tls_cacert = /etc/pki/ca-trust/source/anchors/ldapserver.crt
ldap_id_use_start_tls = false
cache_credentials = true
enumerate = true
ldap_default_bind_dn = cn=admin,dc=rs,dc=local
ldap_default_authtok = Rish@bh123

[nss]
homedir_substring = /home
filter_groups = root
filter_users = root

[pam]
offline_credentials_expiration = 60

[sudo]
```

Restart SSSD

```bash
systemctl restart sssd
```

```bash
sssctl cache-expire -E
```

Test LDAPS authentication

```bash
getent passwd it1
```

```bash
su - it1
```

### Optional: Install GUI
---
Install GUI

```bash
yum group install "Server with GUI"
```

Set graphical target

```bash
systemctl set-default graphical.target
```

```bash
systemctl isolate graphical.target
```

Install VirtualBox Guest Additions (if running in VirtualBox)
Insert Guest Additions CD in VirtualBox

```bash
dnf install -y kernel-headers kernel-devel
```

```bash
mount /dev/sr0 /mnt/
```

```bash
/mnt/VBoxLinuxAdditions.run
```

## Testing and Verification
---
### DNS Testing
---
From client

```bash
dig centos.rs.local
```

```bash
dig wordpress.rs.local
```

```bash
dig gitlab.rs.local
```

### LDAP/LDAPS Authentication
---
Test user authentication

```bash
ssh it1@c1.rs.local
```

Password: It1@123

Test FTP access

```bash
ftp centos.rs.local
```

Username: hr1
Password: Hr1@123

### Web Services Testing
---
- **WordPress**: https://wordpress.rs.local
  - Admin login: wpadmin / Rish@bh123
- **GitLab**: https://gitlab.rs.local
  - Root login: root / Rish@bh123
  - LDAP user login: it1 / It1@123

### Centralized Home Directory
---
From client, create file as LDAP user

```bash
su - it2
```

```bash
echo "Test file" > test-from-client.txt
```

```bash
exit
```

From server, verify file exists

```bash
ls -la /home/it2/test-from-client.txt
```

### Service Status Verification
---
On server

```bash
systemctl status named slapd httpd vsftpd nfs-server
```

```bash
gitlab-ctl status
```

On client

```bash
systemctl status sssd autofs
```

---
---

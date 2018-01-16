# Ansible role to deploy Odoo server docker from [Tecnativa](https://github.com/Tecnativa/docker-odoo-base)

[![](https://img.shields.io/badge/licence-AGPL--3-blue.svg)](http://www.gnu.org/licenses/agpl "License: AGPL-3")

This role allows you to deploy and start 2 Odoo docker instances on your server : a production instance (in /home/docker/odoo) and a test instance (in /home/docker/odootest/).

You would most probably need to adapt the files in the files/ and templates/ directories to your needs, the ones you find here are the ones used by Le Filament on a small VPS server.

On top of deploying and starting the Odoo instances, the following functionalities are added:
- automatic bank statement retrieval nightly (using an homemade [Weboob](http://weboob.org) docker and cron jobs to run it every night for each bank account)
- backup and restore based on duplicity docker from [Tecnativa](https://github.com/Tecnativa/docker-duplicity) which has been customized for our needs to backup on NFS directory and running it every night using cron, keeping only the latest week incremental backups for 1 month

Prior to running this role, you would need to have docker installed on your server and a traefik proxy (which is the purpose of [this role](https://github.com/remi-filament/ansible_role_docker_server))

This role is not foreseen to be used on a development environment (which is the purpose of [that role](https://github.com/remi-filament/docker_odoo_dev))

In order to use this role, you would need to define the following variables for your server (in hostvars for instance) - Only the names of the variables are provided below (not the values) for these used by this role to properly configure everything, and some examples for list of modules, you may copy this file directly in hostvars and set the variable although we could only encourage you to use an Ansible vault and refer vault variables from here:

```json
## Ansible configuration for connecting to remote host
# IP address of server
ansible_ssh_host: 
# User to be used on server (to which Ansible server public key has been provided)
ansible_ssh_user: 
# Encryped password (for elevating rights / sudo)
ansible_ssh_sudo_pass: 
# Server SSHD port
ansible_ssh_port: 

## Odoo configuration
# PRODUCTION URL
odoo_url: 
# TEST URL
odoo_test_url: 

# Odoo version
odoo_version: "10.0"

# Odoo master password
odoo_master_pass: 
# password for proxy protected pages = same as master password but encoded with bcrypt
# Warning: the bcrypt password needs to be with double $$ each time or Docker would not start
PROXY_BCRYPT: 

# Odoo database user
odoo_db_user: 
# Odoo database password
odoo_db_pass: 
# Odoo PROD database name
odoo_db: 
# Odoo TEST database name
odoo_db_test: 

## OCA modules
odoo_custom_modules_oca:
  - repo: server-tools
    modules:
     - date_range
  - repo: bank-statement-import
    modules:
     - account_bank_statement_import_ofx
  - repo: account-financial-reporting
    modules:
     - account_tax_balance
  - repo: knowledge
    modules:
     - document_page
     - document_page_approval
     - document_url
     - knowledge
  - repo: partner-contact
    modules:
     - partner_firstname
  - repo: social
    modules:
     - mail_restrict_follower_selection

## Custom modules
# Where to retrieve them from - for bitbucket
lf_bitbucket_name:
lf_bitbucket_pass:
# List of custom modules to be retrieved
odoo_custom_modules:
 - lefilament_maintenance
 - lefilament_projets

## Mail server configuration - for Odoo
# Mail domain
mailname: 
# Mail server
mailserver: 
# SMTP port
smtpport: 
# SMTP user
smtpuser: 
# SMTP password
smtppass: 

## OPTIONAL - Bank configuration - for Odoo automatic retrieval of statements
# Should auto retrieval be activated ?
banking: yes
# Bank name
bank: 
# Bank website
bank_website: 
# Bank user
bank_login: 
# Bank password
bank_pass: 
# Bank user id
bank_userid: 
# Bank account
bank_account: 
# Bank account 2
bank_account2: 
## 2nd Bank
# Bank 2 name
bank2: 
# Bank 2 authentication type
bank2_authtype: 
# Bank 2 user
bank2_login: 
# Bank 2 password
bank2_pass: 
# Bank 2 account
bank2_account: 
# Bank 2 account 2
bank2_account2: 
```




# Credits

## Contributors

* Remi Cazenave <remi-filament>


## Maintainer

[![](https://le-filament.com/img/logo-lefilament.png)](https://le-filament.com "Le Filament")

This role is maintained by Le Filament

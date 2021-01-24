# Ansible role to deploy Odoo server docker

[![](https://img.shields.io/badge/licence-AGPL--3-blue.svg)](http://www.gnu.org/licenses/agpl "License: AGPL-3")

This role is originally derivated from [Tecnativa/Doodba](https://github.com/Tecnativa/doodba), although it has been reworked lately in order to reduce drastically the size of the Docker image used, since using Tecnativa Doodba, we were reaching 1.5 GB for the image, when the new images are around 800 MB.

This role is taking advantage of [Le Filament Odoo Docker](https://hub.docker.com/repository/docker/lefilament/odoo) which is also described on corresponding [Le Filament GitHub page](https://github.com/lefilament/odoo_docker) and adds on top additional OCA modules and private modules defined with variables *odoo_custom_modules_oca* and *odoo_custom_modules* (see below)

This role allows you to deploy and start 2 Odoo docker instances on your server : a production instance (in /home/docker/odoo) and a test instance (in /home/docker/odootest/).

You would most probably need to adapt the files in the files/ and templates/ directories to your needs, the ones you find here are the ones used by Le Filament on a small VPS server.

On top of deploying and starting the Odoo instances, the following functionalities are added:
- automatic bank statement retrieval nightly (using an homemade [Weboob](http://weboob.org) docker and cron jobs to run it every night for each bank account)
- backup and restore based on duplicity docker from [Tecnativa](https://github.com/Tecnativa/docker-duplicity) which has been customized for our needs to backup on Swift Storage and running it every night using cron, keeping only the latest week incremental backups for 3 weeks
- whitelists based on white list docker from [Tecnativa](https://github.com/Tecnativa/docker-whitelist) in order to block any not identified traffic towards the outside of the server

Prior to running this role, you would need to have docker and docker-compose installed on your server and a traefik proxy (which is the purpose of [this role](https://github.com/lefilament/ansible_role_docker_server))

This role is not foreseen to be used on a development environment (which is the purpose of [that role](https://github.com/lefilament/docker_odoo_dev))

In order to use this role, you would need to define the following variables for your server (in hostvars for instance) - Only the names of the variables are provided below (not the values) for these used by this role to properly configure everything, and some examples for list of modules, you may copy this file directly in hostvars and set the variable although we could only encourage you to use an Ansible vault and refer vault variables from here:

This role is included in [Le Filament full Ansible playbook](https://github.com/lefilament/ansible)

```json
## Ansible configuration for connecting to remote host
# IP address
ansible_ssh_host: "{{ SERVER_srv_ip }}"
# User
host_user: "{{ SERVER_srv_user }}"
# User password
host_password: "{{ SERVER_srv_pass }}"
# Admin User
host_admin_user: "{{ SERVER_srv_admin_user }}"
# Admin password
host_admin_password: "{{ SERVER_srv_admin_pass }}"

# OPTIONAL 2nd USER
# host_user2: "{{ SERVER_srv_user2 }}"
# host_password2: "{{ SERVER_srv_pass2 }}"
# host_user2_pubkey: "{{ SERVER_srv_user2_pubkey }}"

# password for proxy protected pages
srv_proxy_pass: "{{ SERVER_srv_proxy_pass }}"

## SFTP user to connect to Backup server
backup_sftp_user: "{{ SERVER_backup_sftp_user }}"

## Odoo configuration
  
# By default, an Odoo server is deployed with both prod and test instances.
# By default, all variables for test instance are copied from prod one, but URL and database name
odoo_prod:
    # PROD URL
    url: "{{ SERVER_odoo_url }}"
    master_pass: "{{ SERVER_odoo_master_pass }}"
    # Database identifiers user and password
    db_user: "{{ SERVER_odoo_db_user }}"
    db_pass: "{{ SERVER_odoo_db_pass }}"
    # PROD Database name
    db: "{{ SERVER_odoo_db_name }}"
    # Image name to be generated (default = odoo_version)
    image_version: "{{ odoo_version }}"
    # Custom modules Le Filament (one module per repo)
    custom_modules:
     - automatic_bank_statement_import
     - lefilament_account
     - lefilament_export_journal
     - lefilament_generic_reports
    # Custom modules Le Filament (one module per repo and branch is specified instead of taking odoo_version by default)
    custom_modules_branch:
     - repo: lefilament_account
       branch: "{{ odoo_version }}"
    # OCA modules - these should be limited to the ones not already defined
    # in groups_vars/docker_odoo in default_odoo_custom_modules_oca (since these
    # are already part of lefilament/odoo:10.0 and 12.0 dockers)
    custom_modules_oca:
      - repo: server-tools
        modules:
         - auto_backup
      - repo: knowledge
        modules:
         - document_page_approval
      - repo: social
        modules:
         - mail_attach_existing_attachment
      - repo: website-cms
        modules:
         - cms_delete_content
         - cms_form
         - cms_info
         - cms_status_message
    # Other Odoo modules where git repo is the module
    other_repos:
     - repo: filament
       url: git@github.com:lefilament/link_sale_project_tasks.git
    # Other Odoo modules where git repo contains various modules
    other_modules:
     - repo: filament
       url: https://github.com/lefilament/bank-statement-import.git
       branch: 12.0-mig-account_bank_statement_import_ofx
       modules:
         - account_bank_statement_import_ofx
    extra_urls:
      - "sso.le-filament.com"

odoo_nonprod_instances:
      - name: odoo_test
        # Directory where this test instance will be installed on server
        dir: "odootest"
        # TEST URL
        url: "{{ SERVER_odoo_test_url }}"
        # OPTIONAL for when prod instance not deployed on server
        # Hostname of server where prod server is installed (required for retrieving prod database on test ones)
        prod_inv_name: SERVER
        master_pass: "{{ odoo_prod.master_pass }}"
        db_user: "{{ odoo_prod.db_user }}"
        db_pass: "{{ odoo_prod.db_pass }}"
        # TEST Database Name
        db: "{{ SERVER_odoo_db_name_test }}"
        image_version: "{{ odoo_image_tag | default(odoo_version) }}"
        custom_modules: "{{ odoo_prod.custom_modules | default([]) }}"
        custom_modules_branch: "{{ odoo_prod.custom_modules_branch | default([]) }}"
        custom_modules_oca: "{{ odoo_prod.custom_modules_oca | default([]) }}"
        other_repos: "{{ odoo_prod.other_repos | default([]) }}"
        other_modules: "{{ odoo_prod.other_modules | default([]) }}"
        extra_urls:
          - "sso.le-filament.com"

# OPTIONAL - Odoo multilingual - Will install Odoo with all languages (English and French only if set to no)
odoo_multilingual: no


## OPTIONAL - Mail server configuration - for Odoo
# Mail domain
mailname: "{{ SERVER_mail_domain }}"
# Mail server
mailserver: "{{ SERVER_mail_srv_url }}"
# SMTP port
smtpport: 465
# SMTP user
smtpuser: "{{ SERVER_mail_odoo_user }}"
# SMTP password
smtppass: "{{ SERVER_mail_odoo_pass }}"

## OPTIONAL - Bank configuration - for Odoo automatic retrieval of statements
# Should auto retrieval be activated ?
banking: yes
# Bank name
bank: "{{ SERVER_bank }}"
# Bank website
bank_website: "{{ SERVER_bank_website }}"
# Bank user
bank_login: "{{ SERVER_bank_login }}"
# Bank password
bank_pass: "{{ SERVER_bank_pass }}"
# Bank user id
bank_userid: "{{ SERVER_bank_userid }}"
# Bank account
bank_account: "{{ SERVER_bank_account }}"
# Bank account 2
bank_account2: "{{ SERVER_bank_account2 }}"

# OPTIONAL - GIT private keys - for retrieving private repos (outside Le Filament ones)
git_private_keys: "{{ SERVER_git_private_keys }}"


## Backup Swift Storage configuration
swift_odoo_username:
swift_odoo_password:
swift_odoo_authurl:
swift_odoo_authversion:
swift_odoo_tenantname:
swift_odoo_tenantid:
swift_odoo_regionname:

```

# Procedure to restore Odoo from backup

In order to restore Owncloud database and files from backup, it is necessary to first delete the database (from Odoo interface URL/web/database/manager) then change directory to /home/docker/backups and run the following command:
`docker-compose -f backup-odoo.yaml run --rm backup_odoo sh -c "restore --force && createdb -T template0 \$PGDATABASE && pg_restore -d \$PGDATABASE \$SRC/\$PGDATABASE.pgdump"`


You can also copy production database inside test one, be careful with that command that will delete test database at first : change directory to /home/docker/backups and run the following command:
`docker-compose -f restore-odootest.yaml run --rm restore_test`


# Credits

## Contributors

* Remi Cazenave <remi-filament>


## Maintainer

[![](https://le-filament.com/img/logo-lefilament.png)](https://le-filament.com "Le Filament")

This role is maintained by Le Filament

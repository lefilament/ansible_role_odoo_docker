docker_odoo
===========

This role deploys Odoo (community edition) in a Docker, together with PostgreSQL, postfix proxy, metabase, etc. as needed
By default, this role would deploy :
* 1 production instance
* 1 test instance
with the following workflow :
1. Build Odoo image on test instance
2. Test image on test instance
3. Once confirmed OK, copy test image in prod image and restart prod instance on this new image
4. Use prod image on production instance

This role is originally derivated from the work done by [Tecnativa/Doodba](https://github.com/Tecnativa/doodba) for creating a generic Odoo Docker, although it has been reworked lately in order to reduce drastically the size of the Docker image used, since using Tecnativa Doodba, we were reaching 1.5 GB for the image, when the new images are around 800 MB.

This role is taking advantage of [Le Filament Odoo Docker](https://hub.docker.com/repository/docker/lefilament/odoo) which is also described on corresponding [Le Filament GitHub page](https://github.com/lefilament/odoo_docker) and adds on top additional OCA modules and private modules defined with variables *odoo_custom_modules_oca* and *odoo_custom_modules* (see below)


Requirements
------------

None

Role Variables
--------------
This role makes use of an important number of variables.
Part of these variables are described with comments in defaults/main.yml.

Only the most important ones that would change what will be deployed will be further documented below :
* Instances : This role allows for deployment on a server of :
  * 0 or 1 production instance : odoo_prod
  * 0 to x instances of non production instances (test, dev, preprod, etc.) : odoo_nonprod_instances
  The default uncommented values in defaults/main.yml are for 1 prod and 1 test instance on the same server.
  Should you want to change these, you would need
  * to uncomment the variables in odoo_prod if there are no test instances
  * to uncomment the variables in each odoo_nonprod_instances if there are no prod instances
  * in case the prod instance is deployed on another server, you need to fill the inventory_hostname of that server in prod_inv_name variable in each odoo_nonprod_instances

The variable structure for instances is defined below :
```yaml
# Odoo server is deployed with 0 or 1 prod and x test instances.
# You can define as many non prod instances as you want
# You can also deploy only prod instance or only test instances on one server,
# in that case, you would need to uncomment the OPTIONAL variables below
odoo_prod_example: # To be renamed odoo_prod
    # PROD URL (only sub.domain without https:// in front)
    url: "odoo.example.org"
    master_pass: "notSecureEnoughPasswordToBeModified"
    # Database identifiers user and password
    db_user: "odoo"
    db_pass: "odooDbPasswordToBeModified"
    # PROD Database name
    db: "odoo"
    ## OPTIONAL - use this variable to force using multiprocessing iso multithreading
    # force_odoo_workers: True
    ## OPTIONAL - only needed if no nonprod_instances are defined
    ## Custom modules Le Filament (one module per repo)
    # custom_modules:
    #     - automatic_bank_statement_import
    #     - lefilament_account
    #     - lefilament_export_journal
    #     - lefilament_generic_reports
    ## OPTIONAL - only needed if no nonprod_instances are defined
    ## Custom modules Le Filament (one module per repo and branch is specified instead of taking odoo_version by default)
    # custom_modules_branch:
    #     - repo: lefilament_account
    #       branch: "{{ odoo_version }}"
    ## OPTIONAL - only needed if no nonprod_instances are defined
    ## OCA modules - these should be limited to the ones not already defined
    ## in groups_vars/docker_odoo in default_odoo_custom_modules_oca (since these
    ## are already part of lefilament/odoo:10.0, 12.0 and 14.0 dockers)
    # custom_modules_oca:
    #     - repo: server-tools
    #       modules:
    #           - auto_backup
    #     - repo: knowledge
    #       modules:
    #           - document_page_approval
    #     - repo: social
    #       modules:
    #           - mail_attach_existing_attachment
    #     - repo: website-cms
    #       modules:
    #           - cms_delete_content
    #           - cms_form
    #           - cms_info
    #           - cms_status_message
    ## OPTIONAL - only needed if no nonprod_instances are defined
    ## Other Odoo modules where git repo is the module
    # other_repos:
    #     - repo: filament
    #       url: git@github.com:lefilament/link_sale_project_tasks.git
    ## OPTIONAL - only needed if no nonprod_instances are defined
    ## Other Odoo modules where git repo contains various modules
    # other_modules:
    #     - repo: filament
    #       url: https://github.com/lefilament/bank-statement-import.git
    #       branch: 12.0-mig-account_bank_statement_import_ofx
    #       modules:
    #           - account_bank_statement_import_ofx
    ## OPTIONAL extra_urls to be accessible from this Odoo intance only
    ## (if URLs need to be accessible from both prod and non-prod instances, use whitelisted_urls instead)
    # extra_urls:
    #     - url: "docs.example.org"
    #       port: 443
    ## OPTIONAL parameters for deploying another app (for instance a JS app)
    # extra_app:
    #     - name: odoo_app
    #     - image: nginx:latest
    #     - url: app.example.org

odoo_nonprod_instances_example: # To be renamed odoo_nonprod_instances
    - name: odoo_test
      ## Directory where this test instance will be installed on server
      dir: "odootest"
      ## TEST URL (only sub.domain without https:// in front)
      url: "odoo-test.example.org"
      ## OPTIONAL - for when prod instance not deployed on server
      ## Hostname of server where prod server is installed (required for retrieving prod database on test ones)
      # prod_inv_name: OdooProdServer
      ## OPTIONAL - only needed when odoo_prod is not defined
      # master_pass: "notSecureEnoughPasswordToBeModified"
      # db_user: "odoo"
      # db_pass: "odooDbPasswordToBeModified"
      ## TEST Database Name
      db: "odoo_test"
      ## Tag that will be used on the built test Docker image
      image_version: "{{ odoo_version }}_test"
      ## OPTIONAL - use this variable to force using multiprocessing iso multithreading
      # force_odoo_workers: True
      ## Custom modules Le Filament (one module per repo)
      custom_modules:
          - automatic_bank_statement_import
          - lefilament_account
          - lefilament_export_journal
          - lefilament_generic_reports
      ## Custom modules Le Filament (one module per repo and branch is specified instead of taking odoo_version by default)
      custom_modules_branch:
          - repo: lefilament_account
            branch: "{{ odoo_version }}"
      ## OCA modules - these should be limited to the ones not already defined
      ## in groups_vars/docker_odoo in default_odoo_custom_modules_oca (since these
      ## are already part of lefilament/odoo:10.0 and 12.0 dockers)
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
      ## Other Odoo modules where git repo is the module
      other_repos:
          - repo: filament
            url: git@github.com:lefilament/link_sale_project_tasks.git
      ## Other Odoo modules where git repo contains various modules
      other_modules:
          - repo: filament
            url: https://github.com/lefilament/bank-statement-import.git
            branch: 12.0-mig-account_bank_statement_import_ofx
            modules:
                - account_bank_statement_import_ofx
      extra_urls:
          - "docs-test.example.org"
      ## OPTIONAL parameters for deploying another app (for instance a JS app)
      # extra_app:
      #     - name: odootest_app
      #     - image: nginx:latest
      #     - url: app-test.example.org
```
Also backups are designed to be performed only on prod instances, backups can however be restored on every non prod instance.

* Extra modules : although Odoo comes with a number of apps, the real added value comes from community modules. A number of variables allow to retrieve modules from various sources :
  * git_modules_privkey : private key used to retrieve modules from private repos (for now only supports ed25519)
  * custom_modules : allows to retrieve unitary modules from custom_modules_base_url variable (these will use the same branch as odoo_version variable). This is useful if like Le Filament you are developping a number of modules, this way you only have to define path to the modules, and not the full URL each time
  * custom_modules_branch : allows to retrieve unitary modules from custom_modules_base_url variable on a specific git branch
  * custom_modules_oca: this allows to retrieve named modules from OCA repositories (https://github.com/OCA)
  * other_repos : this allows to retrieve modules from a git repository (each repository is a single module)
  * other_modules : this allows to retrieve named modules from git repositories (each repo containing multiple modules)

* Internet Access : by default, the dockers deployed by this role do not have access to Internet (meaning they will not be able to communicate towards the Internet, nor be accessible from the Internet). This is done by creating only internal networks. The following variables allow to either deactivate this isolation, or to whitelists URL that would be accessible from the Odoo Docker instance using [Tecnativa Whitelist Docker](https://github.com/Tecnativa/docker-whitelist) :
  * restrict_internet_access : by setting this variable to true, all deployed Docker will be allowed to initiate connections towards the Internet
  * whitelisted_urls : this list may contain URLs that would be accessible from prod and nonprod Odoo instances
  * extra_urls defined in either odoo_prod or any odoo_nonprod_instances may contain URLs that would be accessible only from the instance in which it is defined (as an example, with this variable you would be able to allow access to some API only for prod instance)

* Mail server : by default, nonprod instances are not allowed to communicate towards mail server and are connected to a [mailhog]() instance allowing to check what e-mails would have been sent out. This would allow again to have only prod instance being able to send e-mails to partners. The following variables can be used to authorize access to mail server (SMTP/IMAP) and to configure local postfix relay (this way you would not need to configure e-mail server credentials in Odoo database) :
  * mailname : if defined (together with the mailserver and smtp** variables) a postfix local relay will be deployed and used by prod instance only. If not defined, a mailhog instance would be deployed on prod also.
  * imap_mailserver : if defined a whitelist is created to allow connecting to IMAP server on port 993 (see also the previous section related to Internet Access for explaining why these whitelists might be required)

* Bank : some variables may be defined to deploy [woob](https://woob.tech/) on the server together with configuration of bank accounts. This would allow to web scrap your bank website to collect daily bank statements that can then be automatically imported in Odoo prod instance.


* Backups (for backups to be deployed, host needs to be in maintenance_contract group) : the backups make use of [Tecnativa Duplicity Docker](https://github.com/Tecnativa/docker-duplicity)
  * swift parameters for 2 object storage instances where backups should be pushed daily
  * odoo_backup_pass : Passphrase for encryption of backups

* Metabase : This role allows for deployment of Metabase, a Business Intelligence (BI) Open Source tool that would be connected with readonly used on prod database in order to extract metrics and indicators from Odoo database and display those in dashboards.


Dependencies
------------

This role requires the following Ansible collection :
* community.docker

This Docker role supposes that Traefik is deployed as an inverseproxy in front of the deployed Dockers.
The following role is used by Le Filament for deploying Traefik : docker_server (https://sources.le-filament.com/lefilament/ansible-roles/docker_server)

Example Playbook
----------------

Given the number of variables (see defaults/main.yml), it would be preferable to create a host_vars file listing all the variables needed for you Odoo server, rather than giving your variables through the playbook directly.

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: docker_odoo }
      vars:
         - { odoo_db_version: 13 }
         - { odoo_version: "14.0" }
         - { odoo_prod: "{{ odoo_prod_example }}" }
         - { odoo_nonprod_instances: "{{ odoo_nonprod_instances_example }}" }

License
-------

AGPL-3

Author Information
------------------

Le Filament (https://le-filament.com)

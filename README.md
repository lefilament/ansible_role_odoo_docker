docker_odoo
===========

This role deploys Odoo (community edition) in a Docker, together with PostgreSQL, postfix proxy, metabase, etc. as needed
By default, this role would deploy as many instances as defined in odoo_instances variable.
Usually we deploy for each of our customers 2 instance (prod and test) with the following workflow :
1. Build Odoo image on test instance
2. Test image on test instance
3. Once confirmed OK, copy test image in prod image and restart prod instance on this new image
4. Use prod image on production instance

This role is originally derivated from the work done by [Tecnativa/Doodba](https://github.com/Tecnativa/doodba) for creating a generic Odoo Docker, although it has been reworked lately in order to reduce drastically the size of the Docker image used, since using Tecnativa Doodba, we were reaching 1.5 GB for the image, when the new images are around 1 GB.

This role is taking advantage of [Le Filament Odoo Docker image](https://hub.docker.com/repository/docker/lefilament/odoo) which is also described on (and built from) [Le Filament GitLab page](https://sources.le-filament.com/lefilament/odoo_docker) and adds on top additional OCA modules and private modules that are used by most of our customers.


Requirements
------------

None

Role Variables
--------------

This role makes use of an important number of variables.
Part of these variables are described with comments in defaults/main.yml.

The variable structure for instances is defined below :
```yaml
# You can define as many non prod instances as you want
# You can also deploy only prod instance or only test instances on one server,
# in that case, you would need to uncomment the OPTIONAL variables below
# By default, an Odoo server is deployed with both prod and test instances.
# By default, all variables for test instance are copied from prod one, but URL and database name

# Odoo instances definition.
odoo_instances:
    # Production instance
    odoo:
        # Name of a production instance. Itself if it's a production instance (like here).
        prod_instance: odoo
        # Name of an instance from which take image (like here). Itself if image build is done on this instance.
        image_instance: odootest
        # Name of an instance from which take backups. Itself if backups are made for this instance (like here).
        backup_instance: odoo
        ## URL (only sub.domain without https:// in front)
        url: "{{ SERVER_odoo_url }}"
        master_pass: "{{ SERVER_odoo_master_pass }}"
        ## Database identifiers user and password
        db_user: "{{ SERVER_odoo_db_user }}"
        db_pass: "{{ SERVER_odoo_db_pass }}"
        ## Database name
        db: "{{ SERVER_odoo_db_name }}"
        ## OPTIONAL - For maintenance only - Backup password
        # odoo_backup_pass: "{{ SERVER_odoo_backup_pass }}"

    # Test instance
    odootest:
        # This test instance rely on previous production instance.
        prod_instance: odoo
        # This instance build it's image.
        image_instance: odootest
        # This instance need to restore backups from production instance.
        backup_instance: odoo
        ## URL (only sub.domain without https:// in front)
        url: "{{ SERVER_odoo_test_url }}"
        ## Database Name
        db: "{{ SERVER_odoo_db_name_test }}"
        ## Custom modules Le Filament (one module per repo and specify branch if differ from default odoo version)
        # custom_modules:
        #     - repo: lefilament_account
        #     - repo: lefilament_export_journal
        #       branch: "14.0"
        ## OCA modules - these should be limited to the ones not already defined
        ## in groups_vars/docker_odoo in default_odoo_custom_modules_oca (since these
        ## are already part of lefilament/odoo:10.0 and 12.0 dockers)
        # custom_modules_oca:
        #     - repo: server-tools
        #       modules:
        #           - auto_backup
        ## Other Odoo modules where git repo is the module
        # other_repos:
        #     - repo: filament
        #       url: git@github.com:lefilament/link_sale_project_tasks.git
        ## Other Odoo modules where git repo contains various modules
        # other_modules:
        #     - repo: filament
        #       url: https://github.com/lefilament/bank-statement-import.git
        #       branch: 12.0-mig-account_bank_statement_import_ofx
        #       modules:
        #           - account_bank_statement_import_ofx
        ## OPTIONAL - Extra pip packages to be installed in image
        # odoo_pip_packages: unidecode
        ## OPTIONAL - Odoo multilingual - Will install Odoo with all languages (English and French only if set to no - by default) - uncomment and set to yes if needed
        # odoo_multilingual: false
        ## OPTIONAL - Force usage of Python 3.6 for Odoo 12.0
        # odoo_python36: false

### OPTIONAL - Mail server configuration - for Odoo - uncomment to add mail server
## Mail domain
# mailname: "{{ SERVER_mail_domain }}"
## Mail server
# mailserver: "{{ SERVER_mail_srv_url }}"
## SMTP port
# smtpport: 465
## SMTP user
# smtpuser: "{{ SERVER_mail_odoo_user }}"
## SMTP password
# smtppass: "{{ SERVER_mail_odoo_pass }}"

## OPTIONAL - GIT private keys - for retrieving private repos (outside Le Filament ones) - uncomment and provide keys as needed
# git_private_keys: "{{ SERVER_git_private_keys }}"

## OPTIONAL - Whitelisted URLs allowed to be reached from Odoo SERVER
# whitelisted_urls:
#     - github.com
#     - sources.le-filament.com
```

* Extra modules : although Odoo comes with a number of apps, the real added value comes from community modules. A number of variables allow to retrieve modules from various sources :
  * git_modules_privkey : private key used to retrieve modules from private repos (for now only supports ed25519)
  * custom_modules : allows to retrieve unitary modules from custom_modules_base_url variable. It's possible to specify the git branch to use (with `branch: <branch>`), else the same branch than odoo version for the current instance will be used. This is useful if like Le Filament you are developping a number of modules, this way you only have to define path to the modules, and not the full URL each time
  * custom_modules_oca: this allows to retrieve named modules from OCA repositories (https://github.com/OCA)
  * other_repos : this allows to retrieve modules from a git repository (each repository is a single module)
  * other_modules : this allows to retrieve named modules from git repositories (each repo containing multiple modules)

* Internet Access : by default, the dockers deployed by this role do not have access to Internet (meaning they will not be able to communicate towards the Internet, nor be accessible from the Internet). This is done by creating only internal networks. The following variables allow to either deactivate this isolation, or to whitelists URL that would be accessible from the Odoo Docker instance using [Tecnativa Whitelist Docker](https://github.com/Tecnativa/docker-whitelist) :
  * restrict_internet_access : by setting this variable to true, all deployed Docker will be allowed to initiate connections towards the Internet
  * whitelisted_urls : this list may contain URLs that would be accessible from prod and nonprod Odoo instances
  * extra_urls defined in either odoo_prod or any odoo_nonprod_instances may contain URLs that would be accessible only from the instance in which it is defined (as an example, with this variable you would be able to allow access to some API only for prod instance)

* Mail server : by default, nonprod instances are not allowed to communicate towards mail server and are connected to a [mailhog](https://github.com/mailhog/MailHog) instance allowing to check what e-mails would have been sent out. This would allow again to have only prod instance being able to send e-mails to partners. The following variables can be used to authorize access to mail server (SMTP/IMAP) and to configure local postfix relay (this way you would not need to configure e-mail server credentials in Odoo database) :
  * mailname : if defined (together with the mailserver and smtp** variables) a postfix local relay will be deployed and used by prod instance only. If not defined, a mailhog instance would be deployed on prod also.
  * imap_mailserver : if defined a whitelist is created to allow connecting to IMAP server on port 993 (see also the previous section related to Internet Access for explaining why these whitelists might be required)

* Backups (for backups to be deployed, host needs to be in maintenance_contract group) : the backups make use of [Tecnativa Duplicity Docker](https://github.com/Tecnativa/docker-duplicity)
  * swift_accounts dict parameters for object storage instances where backups should be pushed daily
  * odoo_backup_pass : Passphrase for encryption of backups

* Metabase : This role allows for deployment of [Metabase](https://www.metabase.com/), a Business Intelligence (BI) Open Source tool that would be connected with readonly used on prod database in order to extract metrics and indicators from Odoo database and display those in dashboards.


Dependencies
------------

This role requires the following Ansible collection :
* community.docker

This Docker role supposes that Traefik is deployed as an inverseproxy in front of the deployed Dockers.
The following role is used by Le Filament for deploying Traefik : docker_server (https://sources.le-filament.com/lefilament/ansible-roles/docker_server)

Example Playbook
----------------

Given the number of variables (see defaults/main.yml), it would be preferable to create a host_vars file listing all the variables needed for you Odoo server, rather than giving your variables through the playbook directly.


License
-------

AGPL-3

Author Information
------------------

Le Filament (https://le-filament.com)

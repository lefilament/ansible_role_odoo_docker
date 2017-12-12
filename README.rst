.. image:: https://img.shields.io/badge/licence-AGPL--3-blue.svg
   :target: http://www.gnu.org/licenses/agpl
   :alt: License: AGPL-3


======================================================================
Le Filament - Ansible role to deploy Odoo server docker from Tecnativa
======================================================================

In order to use this role, you would need to define the following variables for your server (in hostvars for instance):

OCA modules in use in the following format:
OE_CUSTOM_OCA:
  - repo: server-tools
    modules:
     - auto_backup
     - date_range
  - repo: bank-statement-import
    modules:
     - account_bank_statement_import_ofx
  - repo: account-financial-reporting
    modules:
     - account_tax_balance

You can also use private repos (in our example from BitBucket and are retrieved using lf_bitbucket_name and lf_bitbucket_pass over HTTPS) :
OE_CUSTOM:
 - lefilament_maintenance
 - lf_theme

# URL for setting certificate
OE_URL: example.mycompany.com
OE_TEST_URL: testexample.mycompany.com

# Odoo vesion and daemon name
OE_VERSION: "10.0"

# Odoo Master password
OE_SUPERADMIN: adminpassword 
# Bcrypted password - Get this with `htpasswd -nbB odoo $OE_SUPERADMIN`
PROXY_BCRYPT: $2y$05$BWt6wfwC4hTOpxLPoZQUJO1/m53aWpFsCksQB.PXcoWGsYqL4Fsny

# Mail server configuration
mailname: mycompany.com
mailserver: mail.mycompany.com
smtpport: 465
smtpuser: myuser@mycompany.com
smtppass: mypassword


Credits
=======

Contributors ------------

* Remi Cazenave <remi@le-filament.com>


Maintainer ----------

.. image:: https://le-filament.com/images/logo-lefilament.png
   :alt: Le Filament
   :target: https://le-filament.com

This module is maintained by Le Filament

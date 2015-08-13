odoo-saas-tools
=================
Odoo (OpenERP) addons to provide SaaS

Structure of SaaS system:

* SaaS Portal - main database
* SaaS Servers - technical databases to control client databases. SaaS server create, edit, delete databases. Each SaaS Server can be installed on a separate machine (e.g. VPS)
* SaaS Clients - client database to be used by customers. Each SaaS Client is attached to a SaaS Server.

Features
========

* create SaaS Client databases:
  * manually from SaaS Portal backend
  * by client after choosing subdomain (similar to https://www.odoo.com/page/start ) - module saas_portal_start
  * by client after choosing database template (e.g. template for POS, template for ECommerce etc) with auto-generated subdomain (e.g. demo-12345.odoo.com) - module saas_server_templates
  * by client after singing up - module saas_portal_signup
* prepare templates for new SaaS Client database. You are able to connect to template database, install modules you need, edit configuration, edit access rights for customer etc. Such template database will be exactly what a customer will see after database creating.
* connect to existed SaaS Client database as administrator
* control SaaS Client database from SaaS Portal backend:
  * install, update, delete addons
  * configure parameters (e.g. Max Allowed Users)
* collect information from client databases (count of users, disk space usage, etc.)
* notify customers about news by sending messages to Whole Company messaging group *(under development)*
* show message at the top of a page (e.g. Your free trial will expire in about 4 hours  Register now to add 15 days for free!) *(under development)*

Usage
=====

1. Configure Odoo installation
   * set dbfilter, e.g. *%h*
   * execute commands below to allow create databases with dots in name:

   > cd path/to/odoo

   > sed -i 's/matches="[^"]*"//g' addons/web/static/src/xml/base.xml

   * If you run odoo locally, add domains you are going to use to /etc/hosts. E.g.

   > 127.0.0.1	odoo.local # portal

   > 127.0.0.1	s1.odoo.local # server

   > 127.0.0.1	t1.odoo.local # template

   > 127.0.0.1	t2.odoo.local # template

   > 127.0.0.1	client-x.odoo.local

   > 127.0.0.1	client-y.odoo.local

   > 127.0.0.1	client-z.odoo.local

   * Redirect requests to domains above to localhost:8069 (e.g. via nginx)
   
   * install dependencies:
     * pip install oauthlib

2. Create two databases:

   * Main Database, e.g. **odoo.local**:
     * install saas_portal and saas_portal_* (optional) modules
   * Server Database, e.g. **s1.odoo.local**
     * install saas_server

3. Configure **Server Database**
   * Tick "Technical Features" for admin at Settings/Users/Users - Administrator
     * Refresh page
   * Open Settings/Users/OAuth Providers - SaaS
     * click [Edit]
     * update domain name at "Authentication URL" and "Validation URL", change http to https if needed. E.g.
       * http://**odoo.local**/oauth2/auth
       * http://**odoo.local**/oauth2/tokeninfo
	 * Edit Body according to your brand
	 * click [Save]
     * remember value of Client ID field. It's a database.uuid of SaaS Server.
   

4. Configure **Main Database**:
   * open Settings/Configuration/SaaS Portal Settings
     * set *Base SaaS domain*, e.g. **odoo.local**
     * click Apply (do it even if you didn't make changes)

5. Register Server Database in Main Database
   * open SaaS/SaaS/Servers
     * click [Create]
     * set Database Name, e.g. **s1.odoo.local**
     * fix autogenerated Database UUID to actual one (see previous section)
     * click [Save]
	 * click [Open Server]. Be sure that you allow pop-ups in your browser.
       * log out from SaaS Server
	   * click "Log in via SaaS Portal"
	   * now SaaS Portal has access_token to send commands to SaaS Server

6. Create Plan
   * open Saas/SaaS/Plans
     * click [Create]
     * set Plan's name, e.g. "POS + ECommerce"
     * set SaaS Server
     * set Template DB: type name, e.g. **t1.odoo.local**, and click "Create"
     * click [Save]
     * click [Create Template DB].
     * wait couple minutes while Database is being created.
	 * click [Sync server]

7. Prepare Template Database for Plan
   * open template db, e.g. **t1.odoo.local**
   * install modules that will be used for Plan, e.g. *point_of_sale*, *website_sale*
   * make any other changes in database if needed. E.g. configure
     chart of accounts.
   * open Settings/Users/Users - onwer_template. Configure Access Rights for Owner.
	 
8. Try to create database from template
   * open SaaS/Saas/Plans - select plan
     * click [Create Client]
     * set DB Name, e.g. client-x.odoo.local
     * click [Create]
     * wait couple minutes while Database is being created.
	 * click [Sync server]
   * open SaaS/SaaS/Client - choose client
     * click [Configure]
	   * open Parameters tab
	   * add parameter "Max Users", set Value 2
	   * click [Execute]
	   * click [Close]
	 * click [Log in]
	   * you see created database
	   * try to create new user to test "Max Users" setting

9. Get more
   * check description of other saas_* modules to get more features
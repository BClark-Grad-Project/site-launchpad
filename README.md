# Deploy SaaS Application Network (Base Component)
Builds a SaaS framework for a Node.js applications on OpenStack using Juju.  

Node.js and MongoDB will be installed for single server applications.  
Relationships between [mongodb charm](http://bazaar.launchpad.net/~charmers/charms/precise/mongodb/trunk/files) and my modified [NGINX Proxy charm](https://github.com/BClark-Grad-Project/nginx-proxy) 
are permitted for scaling out services.

A prescribed Node.js baseline application [site-template](https://github.com/BClark-Grad-Project/site-template) can be used to begin creating service orchestration.
 
# Using this charm
First, edit `metadata.yaml` > `name` setting if creating more than one SaaS.

Edit `config.yaml` to add info about your app.

Then deploy some basic services by replacing `site-launchpad` with `name` 
setting in `metadata.yaml`.

    juju deploy site-launchpad 
    juju expose site-launchpad # If not using NGINX Proxy 

Adding a authentication server for user information to be shared in 
SaaS network.

    juju add-relation mongodb auth-db
    juju add-relation auth-db site-launchpad

All other database information will be held on the site-template 
server.  If you would like to use a different server for application 
information add configuration to `hooks/mongodb-relation-changed` in 
the `config_app_host()` function.  This data will be stored to the 
application name in `config.yaml` with the "-db" suffix.

	juju deploy mongodb site-template-db
    juju add-relation site-template-db site-launchpad

Add a proxy server(nginx currently needs a degree of manual configuration).

	juju deploy nginx-proxy
	juju expose nginx-proxy
    juju add-relation site-launchpad:website nginx-proxy:reverseproxy

With a proxy in place you can scale up your app (to 10 nodes for example)

    juju add-unit -n 10 myapp


## Charm configuration

Configurable aspects of the charm are listed in `config.yaml`
and can be set by either editing the default values directly
in the yaml file or passing a `myapp.yaml` configuration
file during deployment

    juju deploy --config ~/myapp.yaml node-app myapp

Some of these parameters are used directly by the charm,
and some are passed through to the node app using `config/config.js`.

# Deploy a site-template networked base.

This is a modified version of Mark Mimms node-app charm.

# Using this charm

First, edit `config.yaml` to add info about your app.

Then deploy some basic services

    juju deploy node-app myapp
    juju deploy mongodb
    juju deploy haproxy

relate them

    juju add-relation mongodb myapp
    juju add-relation myapp haproxy

scale up your app (to 10 nodes for example)

    juju add-unit -n 10 myapp

open it up to the outside world

    juju expose haproxy

Find the haproxy instance's public URL from 

    juju status

(or attach it to an elastic IP via the aws console)
and open it up in a browser.


## What the formula does

During the `install` hook,

- make and install `node`/`npm` from config set version.
- `app_repo` can clone `git//address` or use  local files (`site-launchpad/local`) when set to `local`.
- runs `npm` if your app contains `package.json`
- configures networking if your app contains `config/config.js`
- waits to be joined to a `mongodb` service

when related to a `mongodb` service, the formula

- configures db access if your app contains `config/config.js`
- starts your node app as a service


## Charm configuration

Configurable aspects of the charm are listed in `config.yaml`
and can be set by either editing the default values directly
in the yaml file or passing a `myapp.yaml` configuration
file during deployment

    juju deploy --config ~/myapp.yaml node-app myapp

Some of these parameters are used directly by the charm,
and some are passed through to the node app using `config/config.js`.

## Application configuration

The formula looks for `config/config.js` in your app which
starts off looking something like this

    module.exports = config = {
      "name" : "mynodeapp"
      ,"listen_port" : 8000
      ,"mongo_host" : "localhost"
      ,"mongo_port" : 27017
    }


and gets modified with contextually correct configuration information during
either deployment (via the `install` hook) or relation to another service 
(`relation-changed` hook).

This config can be used from within
your application using snippets like

    ...
    var config = require('./config/config')
    ...
      new mongo.Server(config.mongo_host, config.mongo_port, {}),
    ...
    server.listen(config.listen_port);
    ...

Alternatively you could use a "Procfile" in root directory like this:

    web: node app.js

and then get the environment variables from the running environment like this:

    app.set('port', process.env.PORT);

The defined environment variables are:

    NAME
    PORT
    NODE_ENV
    MONGO_HOST
    MONGO_PORT
    MONGO_REPLSET

## Network access

This charm does not open any public ports itself.
The intention is to relate it to a proxy service like
`haproxy`, which will in turn open port 80 to the outside world.
This allows for instant horizontal scalability.

If your node app is itself a proxy and you want it directly exposed,
this can easily be done by adding 

    open-port $app_port

to the bottom of the `install` hook, and then once your stack
is started, you expose

    juju expose myapp

it to the outside world.

By default, juju services within the same environment
can talk to each other on any port over
internal network interfaces.

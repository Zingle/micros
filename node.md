This document describes best practices for developing Node.js micro-services for
Zingle.

TLS Support
===========
Each Node service should support handling its own TLS termination.  This can be
most easily implemented using the **tlsopt** npm package.  For an example of
using the **tlsopt** package, see Example Service below.

Systemd Socket Activation
=========================
Services should support systemd socket activation.  This can be most easily
implemented using the **systemd** npm packge.  For an example of properly
negotiating whether systemd is being used, see Example Service below.

Rollbar
=======
If using Rollbar, do NOT use the `handleUncaughtExceptions` option.  Node.js
apps should crash on unhandled exceptions or rejections.  Otherwise, resources
can be leaked.

Example Service
===============
The following example is the recommended way of handling the requirements
described in this document.

```js
const tlsopt = require("tlsopt");
const systemd = require("systemd");
const Rollbar = require("rollbar");

const server = tlsopt.createServerSync(requestListener);
const port = process.env.LISTEN_PORT || (server.tls ? 443 : 80);
const socket = process.env.LISTEN_PID ? "systemd" : port;

if (process.env.ROLLBAR_TOKEN) {
    const rollbar = new Rollbar({accessToken: process.env.ROLLBAR_TOKEN});
    rollbar.log("app started");
}

server.listen(socket);

function requestListener(req, res) {
    res.end();
}
```

Configuration
=============
Whenever possible, the service should function with an empty configuration.
Reasonable fallbacks should be in place for non-critical functionality.  For
example, logging to Rollbar can be disabled if not configured.  On the other
hand, it's unlikely the service can function without its DB connection, so it
would be reasonable to require that.

Configuration settings should be environment variables in ALL_CAPS.

Common Settings
---------------
Not all of these settings will be needed for all services, but the service
should use the following names when needed.

#### DATABASE_URL
URL describing MariaDB connection settings.  Example:
mysql://user:pass@host:port/schema

#### JWT_SECRET
Shared secret for JWT authentication and authorization.

#### LISTEN_PID
This variable is used by systemd when the service is being socket activated.
When this variable is present, systemd socket activation should be enabled.
Otherwise, the service should listen on LISTEN_PORT if set, or a default port
if not.

#### LISTEN_PORT
Set TCP port to listen on.  Should be ignored if using systemd socket
activation.  Should default to 443 if using TLS.  It's also a good idea to
default to 80 if not using TLS.

#### REDIS_URL
URL describing Redis connection settings.  Example:
redis://:pass@host:port/database

#### ROLLBAR_TOKEN
Access token for reporting to Rollbar.

#### TLS_CERT
Path to PFX encoded secure certificate.  Use **tlsopt** package to handle this
environment variable.

Makefile
========
The service should support a Makefile with a default target which builds the
project, and a clean target to cleanup the build.  A simple npm example could
be:

```
default: build

build: node_modules

clean:
    rm -fr node_modules

node_modules: package.json package-lock.json
    npm install
    touch $@

.PHONY: default build clean
```

Note: if you are copying and pasting the above example, you'll probably have
to replace spaces with tabs for the Makefile to work.

Tests
=====
Unit tests should be included and a "test" script should be defined in the
**package.json** manifest so that tests can be run with `npm test`.  Code
coverage should cover the *majority* of the code.

In addition, a Makefile target name **test** should be added.

```
test:
    npm test
```

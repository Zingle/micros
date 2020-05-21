This document describes best practices for developing PHP micro-services for
Zingle.

Framework
=========
 * Lumen is recommended
 * Laravel is also OK
 * Symfony should work

No Framework
-----------------
 * public folder
 * front controller named index.php in public folder
 * static assets in public folder
 * front controller should proxy requests for included static assets

Rollbar Support
===============
Should report errors or other useful info to Rollbar.  Each service should have
have its own Rollbar access token.  To generate an access token:

 * login to Rollbar
 * select Projects drop down
 * for live tokens, select Zingle project
 * for dev tokens, select Zingle-Dev project
 * select Settings
 * select Project Access Tokens
 * select Create a new access token
 * grant the token the post_server_item scope
 * give it a useful name
 * configure token within app using the ROLLBAR_TOKEN environment variable.

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

#### REDIS_URL
URL describing Redis connection settings.  Example:
redis://:pass@host:port/database

#### ROLLBAR_TOKEN
Access token for reporting to Rollbar.

#### STORAGE_PATH
Directory where app will write files, such as logs, compiled views, etc.  In
most environments, the application will be mounted read-only, so it will not
be possible to write directly into the project folder.

Makefile
========
The service should support a Makefile with a default target which builds the
project, and a clean target to cleanup the build.  A simple composer example
could be:

```
default: build

build: vendor

clean:
    rm -fr vendor

vendor: composer.json composer.lock
    composer install

.PHONY: default build clean
```

Note: if you are copying and pasting the above example, you'll probably have
to replace spaces with tabs for the Makefile to work.

.. _quick-docker:

Installing using Docker
=======================

With dockerized Weblate deployment you can get your personal Weblate instance
up an running in seconds. All of Weblate's dependencies are already included.
PostgreSQL is set up as the default database.

.. include:: steps/hw.rst

.. _docker-deploy:

Installation
------------

The following examples assume you have a working Docker environment, with
``docker-compose`` installed. Please check the Docker documentation for instructions.

1. Clone the weblate-docker repo:

   .. code-block:: sh

        git clone https://github.com/WeblateOrg/docker-compose.git weblate-docker
        cd weblate-docker

2. Create a :file:`docker-compose.override.yml` file with your settings.
   See :ref:`docker-environment` for full list of environment variables.

   .. code-block:: yaml

        version: '3'
        services:
          weblate:
            ports:
              - 80:8080
            environment:
              WEBLATE_EMAIL_HOST: smtp.example.com
              WEBLATE_EMAIL_HOST_USER: user
              WEBLATE_EMAIL_HOST_PASSWORD: pass
              WEBLATE_SERVER_EMAIL: weblate@example.com
              WEBLATE_DEFAULT_FROM_EMAIL: weblate@example.com
              WEBLATE_ALLOWED_HOSTS: weblate.example.com,localhost
              WEBLATE_ADMIN_PASSWORD: password for the admin user
              WEBLATE_ADMIN_EMAIL: weblate.admin@example.com

   .. note::

        If :envvar:`WEBLATE_ADMIN_PASSWORD` is not set, the admin user is created with
        a random password shown on first startup.
        
        Append ',localhost' to :envvar:`WEBLATE_ALLOWED_HOSTS` to be able to access locally for testing.
        
        You may also need to edit the *docker-compose.yml* file and change the default port from 80 if you already have a web server running on your local machine

3. Start Weblate containers:

   .. code-block:: sh

        docker-compose up

Enjoy your Weblate deployment, it's accessible on port 80 of the ``weblate`` container.

.. versionchanged:: 2.15-2

    The setup has changed recently, priorly there was separate web server
    container, since 2.15-2 the web server is embedded in the Weblate container.

.. versionchanged:: 3.7.1-6

   In July 2019 (starting with the 3.7.1-6 tag), the containers is not running
   as root. As a consequence this has lead to changed exposed port from 80 to
   8080.

.. seealso:: :ref:`invoke-manage`

.. _docker-ssl:

Docker container with HTTPS support
-----------------------------------

Please see :ref:`docker-deploy` for generic deployment instructions, this
section only mentions differences compared to it.

Using own SSL certificates
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 3.8-3

In case you have own SSL certificate you want to use, simply place the files
into the Weblate data volume:

* :file:`ssl/fullchain.pem` containing the certificate including any needed CA certificates
* :file:`ssl/privkey.pem` containing the private key


.. hint::

   The placement of the Docker volume depends on your Docker configuration, but
   usually it is stored in
   :file:`/var/lib/docker/volumes/weblate-docker_weblate-data/_data/`. See
   `Docker volumes documentation <https://docs.docker.com/storage/volumes/>`_
   for more info.

Additionally Weblate container will now accept SSL connections on port 4443,
you will want include the port forwarding for HTTPS in docker compose override:

.. code-block:: yaml

     version: '3'
     services:
       weblate:
         ports:
           - 80:8080
           - 443:4443

Automatic SSL certificates using Let’s Encrypt
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In case you want to use `Let’s Encrypt <https://letsencrypt.org/>`_
automatically generated SSL certificates on public installation, you need to
add a reverse HTTPS proxy an additional Docker container, `https-portal
<https://hub.docker.com/r/steveltn/https-portal/>`_ will be used for that.
This is made use of in the :file:`docker-compose-https.yml` file.  Then create
a :file:`docker-compose-https.override.yml` file with your settings:

.. code-block:: yaml

    version: '3'
    services:
      weblate:
        environment:
          WEBLATE_EMAIL_HOST: smtp.example.com
          WEBLATE_EMAIL_HOST_USER: user
          WEBLATE_EMAIL_HOST_PASSWORD: pass
          WEBLATE_ALLOWED_HOSTS: weblate.example.com
          WEBLATE_ADMIN_PASSWORD: password for admin user
      https-portal:
        environment:
          DOMAINS: 'weblate.example.com -> http://weblate:8080'

Whenever invoking :program:`docker-compose` you need to pass both files to it,
and then do:

.. code-block:: console

    docker-compose -f docker-compose-https.yml -f docker-compose-https.override.yml build
    docker-compose -f docker-compose-https.yml -f docker-compose-https.override.yml up

Upgrading the Docker container
------------------------------

Usually it is good idea to only update the Weblate container and keep the PostgreSQL
container at the version you have, as upgrading PostgreSQL is quite painful and in most
cases does not bring many benefits.

You can do this by sticking with the existing docker-compose and just pull
the latest images and then restart:

.. code-block:: sh

    docker-compose stop
    docker-compose pull
    docker-compose up

The Weblate database should be automatically migrated on first startup, and there
should be no need for additional manual actions.

.. note::

    Upgrades across 3.0 are not supported by Weblate. If you are on 2.x series
    and want to upgrade to 3.x, first upgrade to the latest 3.0.1-x (at time of
    writing this it is the ``3.0.1-7``) image, which will do the migration and then
    continue upgrading to newer versions.

.. _docker-environment:

Docker environment variables
----------------------------

Many of Weblate's :ref:`config` can be set in the Docker container using environment variables:

Generic settings
~~~~~~~~~~~~~~~~

.. envvar:: WEBLATE_DEBUG

    Configures Django debug mode using :setting:`DEBUG`.

    **Example:**

    .. code-block:: yaml

        environment:
          WEBLATE_DEBUG: 1

    .. seealso::

            :ref:`production-debug`.

.. envvar:: WEBLATE_LOGLEVEL

    Configures the logging verbosity.


.. envvar:: WEBLATE_SITE_TITLE

    Configures the site-title shown on the heading of all pages.

.. envvar:: WEBLATE_ADMIN_NAME
.. envvar:: WEBLATE_ADMIN_EMAIL

    Configures the site-admin's name and e-mail.

    **Example:**

    .. code-block:: yaml

        environment:
          WEBLATE_ADMIN_NAME: Weblate admin
          WEBLATE_ADMIN_EMAIL: noreply@example.com

    .. seealso::

            :ref:`production-admins`

.. envvar:: WEBLATE_ADMIN_PASSWORD

    Sets the password for the admin user. If not set, the admin user is created with a random
    password shown on first startup.

    .. versionchanged:: 2.9

        Since version 2.9, the admin user is adjusted on every container
        startup to match :envvar:`WEBLATE_ADMIN_PASSWORD`, :envvar:`WEBLATE_ADMIN_NAME`
        and :envvar:`WEBLATE_ADMIN_EMAIL`.

.. envvar:: WEBLATE_SERVER_EMAIL
.. envvar:: WEBLATE_DEFAULT_FROM_EMAIL

    Configures the address for outgoing e-mails.

    .. seealso::

        :ref:`production-email`

.. envvar:: WEBLATE_ALLOWED_HOSTS

    Configures allowed HTTP hostnames using :setting:`ALLOWED_HOSTS` and sets
    sitename to the first one.

    **Example:**

    .. code-block:: yaml

        environment:
          WEBLATE_ALLOWED_HOSTS: weblate.example.com,example.com

    .. seealso::

        :ref:`production-hosts`,
        :ref:`production-site`

.. envvar:: WEBLATE_SECRET_KEY

    Configures the secret used by Django for cookie signing.

    .. deprecated:: 2.9

        The secret is now generated automatically on first startup, there is no
        need to set it manually.

    .. seealso::

        :ref:`production-secret`

.. envvar:: WEBLATE_REGISTRATION_OPEN

    Configures whether registrations are open by toggling :std:setting:`REGISTRATION_OPEN`.

    **Example:**

    .. code-block:: yaml

        environment:
          WEBLATE_REGISTRATION_OPEN: 0

.. envvar:: WEBLATE_TIME_ZONE

    Configures the used time zone in Weblate, see :std:setting:`django:TIME_ZONE`.

    .. note::

       To change the time zone of the Docker container itself, use the ``TZ``
       environment variable.

    **Example:**

    .. code-block:: yaml

        environment:
          WEBLATE_TIME_ZONE: Europe/Prague

.. envvar:: WEBLATE_ENABLE_HTTPS

    Makes Weblate assume it is operated behind a reverse HTTPS proxy, it makes
    Weblate use HTTPS in e-mail and API links or set secure flags on cookies.

    .. note::

        This does not make the Weblate container accept HTTPS connections, you
        need to configure that as well, see :ref:`docker-ssl` for examples.

    **Example:**

    .. code-block:: yaml

        environment:
          WEBLATE_ENABLE_HTTPS: 1

    .. seealso::

        :ref:`production-site`

.. envvar:: WEBLATE_IP_PROXY_HEADER

    Lets Weblate fetch the IP address from any given HTTP header. Use this when using
    a reverse proxy in front of the Weblate container.

    Enables :setting:`IP_BEHIND_REVERSE_PROXY` and sets :setting:`IP_PROXY_HEADER`.

    .. note::

        The format must conform to Django's expectations. Django
        `transforms <https://docs.djangoproject.com/en/2.2/ref/request-response/#django.http.HttpRequest.META>`_ 
        raw HTTP header names as follows:
        
        - converts all characters to uppercase
        - replaces any hyphens with underscores
        - prepends ``HTTP_`` prefix
        
        So ``X-Forwarded-For`` would be mapped to ``HTTP_X_FORWARDED_FOR``.

    **Example:**

    .. code-block:: yaml

        environment:
          WEBLATE_IP_PROXY_HEADER: HTTP_X_FORWARDED_FOR


.. envvar:: WEBLATE_REQUIRE_LOGIN

    Configures login required for the whole of the Weblate installation using :setting:`LOGIN_REQUIRED_URLS`.

    **Example:**

    .. code-block:: yaml

        environment:
          WEBLATE_REQUIRE_LOGIN: 1

.. envvar:: WEBLATE_LOGIN_REQUIRED_URLS_EXCEPTIONS

    Adds URL exceptions for login required for the whole Weblate installation using :setting:`LOGIN_REQUIRED_URLS_EXCEPTIONS`.

.. envvar:: WEBLATE_GOOGLE_ANALYTICS_ID

    Configures ID for Google Analytics by changing :setting:`GOOGLE_ANALYTICS_ID`.

.. envvar:: WEBLATE_GITHUB_USERNAME

    Configures GitHub username for GitHub pull-requests by changing
    :setting:`GITHUB_USERNAME`.

    .. seealso::

       :ref:`github-push`,
       :ref:`hub-setup`

.. envvar:: WEBLATE_SIMPLIFY_LANGUAGES

    Configures the language simplification policy, see :setting:`SIMPLIFY_LANGUAGES`.

.. envvar:: WEBLATE_AKISMET_API_KEY

    Configures the Akismet API key, see :setting:`AKISMET_API_KEY`.

.. envvar:: WEBLATE_GPG_IDENTITY

   Configures GPG signing of commits, see :setting:`WEBLATE_GPG_IDENTITY`.

   .. seealso::

       :ref:`gpg-sign`

.. envvar:: WEBLATE_URL_PREFIX

   Configures URL prefix where Weblate is running, see :setting:`URL_PREFIX`.


Machine translation settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. envvar:: WEBLATE_MT_DEEPL_KEY

    Enables :ref:`deepl` machine translation and sets :setting:`MT_DEEPL_KEY`

.. envvar:: WEBLATE_MT_GOOGLE_KEY

    Enables :ref:`google-translate` and sets :setting:`MT_GOOGLE_KEY`

.. envvar:: WEBLATE_MT_MICROSOFT_COGNITIVE_KEY

    Enables :ref:`ms-cognitive-translate` and sets :setting:`MT_MICROSOFT_COGNITIVE_KEY`

.. envvar:: WEBLATE_MT_MYMEMORY_ENABLED

    Enables :ref:`mymemory` machine translation and sets
    :setting:`MT_MYMEMORY_EMAIL` to :envvar:`WEBLATE_ADMIN_EMAIL`.

.. envvar:: WEBLATE_MT_GLOSBE_ENABLED

    Enables :ref:`glosbe` machine translation.

Authentication settings
~~~~~~~~~~~~~~~~~~~~~~~

LDAP
++++

.. envvar:: WEBLATE_AUTH_LDAP_SERVER_URI
.. envvar:: WEBLATE_AUTH_LDAP_USER_DN_TEMPLATE
.. envvar:: WEBLATE_AUTH_LDAP_USER_ATTR_MAP
.. envvar:: WEBLATE_AUTH_LDAP_BIND_DN
.. envvar:: WEBLATE_AUTH_LDAP_BIND_PASSWORD

    LDAP authentication configuration.

    **Example:**

    .. code-block:: yaml

        environment:
          WEBLATE_AUTH_LDAP_SERVER_URI: ldap://ldap.example.org
          WEBLATE_AUTH_LDAP_USER_DN_TEMPLATE: uid=%(user)s,ou=People,dc=example,dc=net
          # map weblate 'full_name' to ldap 'name' and weblate 'email' attribute to 'mail' ldap attribute.
          # another example that can be used with OpenLDAP: 'full_name:cn,email:mail'
          WEBLATE_AUTH_LDAP_USER_ATTR_MAP: full_name:name,email:mail

    .. seealso::

        :ref:`ldap-auth`

GitHub
++++++

.. envvar:: WEBLATE_SOCIAL_AUTH_GITHUB_KEY
.. envvar:: WEBLATE_SOCIAL_AUTH_GITHUB_SECRET

    Enables :ref:`github_auth`.

BitBucket
+++++++++

.. envvar:: WEBLATE_SOCIAL_AUTH_BITBUCKET_KEY
.. envvar:: WEBLATE_SOCIAL_AUTH_BITBUCKET_SECRET

    Enables :ref:`bitbucket_auth`.

Facebook
++++++++

.. envvar:: WEBLATE_SOCIAL_AUTH_FACEBOOK_KEY
.. envvar:: WEBLATE_SOCIAL_AUTH_FACEBOOK_SECRET

    Enables :ref:`facebook_auth`.

Google
++++++

.. envvar:: WEBLATE_SOCIAL_AUTH_GOOGLE_OAUTH2_KEY
.. envvar:: WEBLATE_SOCIAL_AUTH_GOOGLE_OAUTH2_SECRET

    Enables :ref:`google_auth`.

GitLab
++++++

.. envvar:: WEBLATE_SOCIAL_AUTH_GITLAB_KEY
.. envvar:: WEBLATE_SOCIAL_AUTH_GITLAB_SECRET
.. envvar:: WEBLATE_SOCIAL_AUTH_GITLAB_API_URL

    Enables :ref:`gitlab_auth`.

Azure Active Directory
++++++++++++++++++++++

.. envvar:: WEBLATE_SOCIAL_AUTH_AZUREAD_OAUTH2_KEY
.. envvar:: WEBLATE_SOCIAL_AUTH_AZUREAD_OAUTH2_SECRET

    Enables Azure Active Directory authentication, see :doc:`psa:backends/azuread`.

Azure Active Directory with Tenant support
++++++++++++++++++++++++++++++++++++++++++

.. envvar:: WEBLATE_SOCIAL_AUTH_AZUREAD_TENANT_OAUTH2_KEY
.. envvar:: WEBLATE_SOCIAL_AUTH_AZUREAD_TENANT_OAUTH2_SECRET
.. envvar:: WEBLATE_SOCIAL_AUTH_AZUREAD_TENANT_OAUTH2_TENANT_ID

    Enables Azure Active Directory authentication with Tenant support, see
    :doc:`psa:backends/azuread`.

Linux vendors
+++++++++++++

You can enable authentication using Linux vendors authentication services by
setting following variables to any value.

.. envvar:: WEBLATE_SOCIAL_AUTH_FEDORA
.. envvar:: WEBLATE_SOCIAL_AUTH_OPENSUSE
.. envvar:: WEBLATE_SOCIAL_AUTH_UBUNTU

Other authentication settings
+++++++++++++++++++++++++++++

.. envvar:: WEBLATE_NO_EMAIL_AUTH

    Disables e-mail authentication when set to any value.


PostgreSQL database setup
~~~~~~~~~~~~~~~~~~~~~~~~~

The database is created by :file:`docker-compose.yml`, so these settings affect
both Weblate and PostgreSQL containers.

.. seealso:: :ref:`database-setup`

.. envvar:: POSTGRES_PASSWORD

    PostgreSQL password.

.. envvar:: POSTGRES_USER

    PostgreSQL username.

.. envvar:: POSTGRES_DATABASE

    PostgreSQL database name.

.. envvar:: POSTGRES_HOST

    PostgreSQL server hostname or IP address. Defaults to ``database``.

.. envvar:: POSTGRES_PORT

    PostgreSQL server port. Defaults to none (uses the default value).

.. envvar:: POSTGRES_SSL_MODE

   Configure how PostgreSQL handles SSL in connection to the server, for possible choices see
   `SSL Mode Descriptions <https://www.postgresql.org/docs/11/libpq-ssl.html#LIBPQ-SSL-SSLMODE-STATEMENTS>`_


Caching server setup
~~~~~~~~~~~~~~~~~~~~

Using Redis is strongly recommended by Weblate and you have to provide a Redis
instance when running Weblate in Docker.

.. seealso:: :ref:`production-cache`

.. envvar:: REDIS_HOST

   The Redis server hostname or IP address. Defaults to ``cache``.

.. envvar:: REDIS_PORT

    The Redis server port. Defaults to ``6379``.

.. envvar:: REDIS_DB

    The Redis database number, defaults to ``1``.

.. envvar:: REDIS_PASSWORD
   
    The Redis server password, not used by default.

Email server setup
~~~~~~~~~~~~~~~~~~

To make outgoing e-mail work, you need to provide a mail server.

.. seealso:: :ref:`out-mail`

.. envvar:: WEBLATE_EMAIL_HOST

    Mail server, the server has to listen on port 587 and understand TLS.

    .. seealso:: :setting:`django:EMAIL_HOST`

.. envvar:: WEBLATE_EMAIL_PORT

    Mail server port. Use if your cloud provider or ISP blocks outgoing
    connections on port 587.

    .. seealso:: :setting:`django:EMAIL_PORT`

.. envvar:: WEBLATE_EMAIL_HOST_USER

    Email authentication user, do NOT use quotes here.

    .. seealso:: :setting:`django:EMAIL_HOST_USER`

.. envvar:: WEBLATE_EMAIL_HOST_PASSWORD

    Email authentication password, do NOT use quotes here.

    .. seealso:: :setting:`django:EMAIL_HOST_PASSWORD`

.. envvar:: WEBLATE_EMAIL_USE_SSL

    Whether to use an implicit TLS (secure) connection when talking to the SMTP
    server. In most e-mail documentation, this type of TLS connection is referred
    to as SSL. It is generally used on port 465. If you are experiencing
    problems, see the explicit TLS setting :envvar:`WEBLATE_EMAIL_USE_TLS`.

    .. seealso:: :setting:`django:EMAIL_USE_SSL`

.. envvar:: WEBLATE_EMAIL_USE_TLS

    Whether to use a TLS (secure) connection when talking to the SMTP server.
    This is used for explicit TLS connections, generally on port 587. If you
    are experiencing connections that hang, see the implicit TLS setting
    :envvar:`WEBLATE_EMAIL_USE_SSL`.

    .. seealso:: :setting:`django:EMAIL_USE_TLS`

Error reporting
~~~~~~~~~~~~~~~

It is recommended to collect errors from the installation in a systematic way,
see :ref:`collecting-errors`.

To enable support for Rollbar, set the following:

.. envvar:: ROLLBAR_KEY

    Your Rollbar post server access token.

.. envvar:: ROLLBAR_ENVIRONMENT

    Your Rollbar environment, defaults to ``production``.

To enable support for Sentry, set following:

.. envvar:: SENTRY_DSN

    Your Sentry DSN.

.. envvar:: SENTRY_PUBLIC_DSN

    Your Sentry public DSN.

.. envvar:: SENTRY_ENVIRONMENT

    Your Sentry environment, defaults to ``production``.

Further configuration customization
-----------------------------------

You can additionally override the configuration in
:file:`/app/data/settings-override.py`. This is executed after all environment
settings are loaded, so it gets completely set up, and can be used to customize
anything.

Hub setup
---------

In order to use the GitHub's pull-request feature, you must initialize hub configuration by entering the Weblate container and executing an arbitrary Hub command. For example:

.. code-block:: sh

    docker-compose exec --user weblate weblate bash
    cd
    HOME=/app/data/home hub clone octocat/Spoon-Knife

The username passed for credentials must be the same as :setting:`GITHUB_USERNAME`.

.. seealso::

    :ref:`github-push`,
    :ref:`hub-setup`

Select your machine - local or cloud providers
----------------------------------------------

With docker-machine you can create your Weblate deployment either on your local
machine, or on any large number of cloud-based deployments on e.g. Amazon AWS,
Greenhost, and many other providers.

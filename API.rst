===
API
===

Configuration
=============
Requires two environment variables set:

``CONFIG_PATH``: Path of the directory with the configuration of the API and
the AWS Federation Proxy. Additionally to the AFP configuration the API itself
needs the following setting:

.. code-block:: yaml

    'api': {
      'user_identification': {
        'environment_field': 'REMOTE_USER'
      }
    },
    'logging_handler': {
      'module': 'logging',
      'class': 'SysLogHander',
      'args': [],
      'kwargs': {}
    }

The ``environment_field`` specifies which field from the WSGI environment identifies
the user, i.e. it is considered to be the username.

For logging, a single logger object is used. But the ``logging_handler`` setting
allows you to add a handler to that logger, so you can send log messages to
the destination of your choice.

``ACCOUNT_CONFIG_PATH``: Path of the directory with the configuration of all
Accounts

**For details in configuration, please see the AFP Section.**

You use the example to set the environment in the context of an apache web
server:

.. code-block:: apache

    <Location /path/to/afp_human>
        SetEnv CONFIG_PATH "/path/to/config_human"
        SetEnv ACCOUNT_CONFIG_PATH "/path/to/account_configuration"
        AuthType Basic
        AuthName "auth_name"
        Require group group_name
        AuthPAM_Enabled On/Off
    </Location>

    <Location /path/to/afp_machine>
        SetEnv CONFIG_PATH "/path/to/config_machine"
        SetEnv ACCOUNT_CONFIG_PATH "/path/to/account_configuration"
    </Location>

    WSGIScriptAlias /path/to/afp_human "/var/www/afp-core/api.wsgi"
    WSGIScriptAlias /path/to/afp_machine "/var/www/afp-core/api.wsgi"

API-Endpoints
=============

Human-Authentication
--------------------

Accounts and Roles
~~~~~~~~~~~~~~~~~~

:Endpoint: ``/account``

Returns a set of all accounts and roles for the current user.

**Returns JSON:**

.. code-block:: json

    {
      "accountname1": ["rolename1", "rolename2"],
      "accountname2": ["rolename3", "rolename4"],
    }

Credentials and Console-URL
~~~~~~~~~~~~~~~~~~~~~~~~~~~

:Endpoint: ``/account/<account>/<role>[?callbackurl=<CallbackURL>]``

Returns a dict of credentials (``access_key``, ``secret_key`` and
``session_token``) and console URL for the specified role in the
specified account.

If ``callbackurl`` is set the user of the console will be redirected
to this URL after the credentials expire.

**Returns JSON:**

.. code-block:: json

    {
      "credentials": {
        "access_key": "AKIAIOSFODNN7EXAMPLE",
        "secret_key": "aJalrXUtnFEMI/K7MDENG/bPxRfiCYzEXAMPLEKEY",
        "session_token": "BQoEXAMPLEH4aoAH0gNCAPyJxz4BlCFFxWNE1OPTgk5TthT+..."
      },
      "console_url": "https://signin.aws.amazon.com/federation?Action=login&..."
    }

Credentials only
~~~~~~~~~~~~~~~~

:Endpoint: ``/account/<account>/<role>/credentials``

Returns a dict of credentials
(``access_key``, ``secret_key`` and ``session_token``).

**Returns JSON:**

.. code-block:: json

    {
      "credentials": {
        "access_key": "AKIAIOSFODNN7EXAMPLE",
        "secret_key": "aJalrXUtnFEMI/K7MDENG/bPxRfiCYzEXAMPLEKEY",
        "session_token": "BQoEXAMPLEH4aoAH0gNCAPyJxz4BlCFFxWNE1OPTgk5TthT+..."
      }
    }

Console-URL only
~~~~~~~~~~~~~~~~

:Endpoint: ``/account/<account>/<role>/consoleurl[?callbackurl=<CallbackURL>]``

Returns a string of the console URL for the specified role in the specified
account.

If ``callbackurl`` the user of the console will be redirected to this URL after
the credentials expire.

**Returns Plaintext:**

::

    https://signin.aws.amazon.com/federation?Action=login&...

Machine-Authentication
----------------------

Rolenames
~~~~~~~~~

:Endpoint: ``/meta-data/iam/security-credentials/``

Return a single rolename.

This endpoint is used to authenticate from Boto. Returns an error,
if the provider does not return a single account/role combination

**Returns Plaintext:**

::

    rolename

Security-Credentials
~~~~~~~~~~~~~~~~~~~~

:Endpoint: ``/meta-data/iam/security-credentials/<rolename>``

Returns dict of credentials
(``access_key``, ``secret_key`` and ``session_token``).

This endpoint is used to authenticate from Boto. Returns an error,
if the provider does not return a single account/role combination
or if the user has no access to the given role.

**Returns JSON:**

.. code-block:: json

    {
      "Code": "Success",
      "LastUpdated": "1970-01-01T00:00:00Z",
      "AccessKeyId": "AKIAIOSFODNN7EXAMPLE",
      "SecretAccessKey": "aJalrXUtnFEMI/K7MDENG/bPxRfiCYzEXAMPLEKEY",
      "Token": "BQoEXAMPLEH4aoAH0gNCAPyJxz4BlCFFxWNE1OPTgk5TthT+...",
      "Expiration": "2038-01-19T03:14:07Z",
      "Type": "AWS-HMAC"
    }

Status
~~~~~~

:Endpoint: ``/status``

Returns a dict of monitoring information (``status``, ``message``)

**Returns JSON:**

.. code-block:: json

    {
      "status": "200",
      "message": "OK"
    }

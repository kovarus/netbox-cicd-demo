# Required Configuration Settings

## ALLOWED_HOSTS

This is a list of valid fully-qualified domain names (FQDNs) that is used to reach the NetBox service. Usually this is the same as the hostname for the NetBox server, but can also be different (e.g. when using a reverse proxy serving the NetBox website under a different FQDN than the hostname of the NetBox server). NetBox will not permit access to the server via any other hostnames (or IPs). The value of this option is also used to set `CSRF_TRUSTED_ORIGINS`, which restricts `HTTP POST` to the same set of hosts (more about this [here](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-CSRF_TRUSTED_ORIGINS)). Keep in mind that NetBox, by default, has `USE_X_FORWARDED_HOST = True` (in `netbox/netbox/settings.py`) which means that if you're using a reverse proxy, it's the FQDN used to reach that reverse proxy which needs to be in this list (more about this [here](https://docs.djangoproject.com/en/1.9/ref/settings/#allowed-hosts)).

Example:

```
ALLOWED_HOSTS = ['netbox.example.com', '192.0.2.123']
```

---

## DATABASE

NetBox requires access to a PostgreSQL database service to store data. This service can run locally or on a remote system. The following parameters must be defined within the `DATABASE` dictionary:

* `NAME` - Database name
* `USER` - PostgreSQL username
* `PASSWORD` - PostgreSQL password
* `HOST` - Name or IP address of the database server (use `localhost` if running locally)
* `PORT` - TCP port of the PostgreSQL service; leave blank for default port (5432)

Example:

```
DATABASE = {
    'NAME': 'netbox',               # Database name
    'USER': 'netbox',               # PostgreSQL username
    'PASSWORD': 'J5brHrAXFLQSif0K', # PostgreSQL password
    'HOST': 'localhost',            # Database server
    'PORT': '',                     # Database port (leave blank for default)
}
```

---

## REDIS

[Redis](https://redis.io/) is an in-memory data store similar to memcached. While Redis has been an optional component of
NetBox since the introduction of webhooks in version 2.4, it is required starting in 2.6 to support NetBox's caching
functionality (as well as other planned features).

Redis is configured using a configuration setting similar to `DATABASE`:

* `HOST` - Name or IP address of the Redis server (use `localhost` if running locally)
* `PORT` - TCP port of the Redis service; leave blank for default port (6379)
* `PASSWORD` - Redis password (if set)
* `DATABASE` - Numeric database ID for webhooks
* `CACHE_DATABASE` - Numeric database ID for caching
* `DEFAULT_TIMEOUT` - Connection timeout in seconds
* `SSL` - Use SSL connection to Redis

Example:

```
REDIS = {
    'HOST': 'localhost',
    'PORT': 6379,
    'PASSWORD': '',
    'DATABASE': 0,
    'CACHE_DATABASE': 1,
    'DEFAULT_TIMEOUT': 300,
    'SSL': False,
}
```

!!! note:
    If you were using these settings in a prior release with webhooks, the `DATABASE` setting remains the same but
    an additional `CACHE_DATABASE` setting has been added with a default value of 1 to support the caching backend. The
    `DATABASE` setting will be renamed in a future release of NetBox to better relay the meaning of the setting. 

!!! warning:
    It is highly recommended to keep the webhook and cache databases seperate. Using the same database number for both may result in webhook
    processing data being lost in cache flushing events.

---

## SECRET_KEY

This is a secret cryptographic key is used to improve the security of cookies and password resets. The key defined here should not be shared outside of the configuration file. `SECRET_KEY` can be changed at any time, however be aware that doing so will invalidate all existing sessions.

Please note that this key is **not** used for hashing user passwords or for the encrypted storage of secret data in NetBox.

`SECRET_KEY` should be at least 50 characters in length and contain a random mix of letters, digits, and symbols. The script located at `netbox/generate_secret_key.py` may be used to generate a suitable key.

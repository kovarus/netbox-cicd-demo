# Migration

!!! warning
    As of version 2.5, NetBox no longer supports Python 2. Python 3 is required to run any 2.5 release or later.

## Ubuntu

Remove the Python2 version of gunicorn:

```no-highlight
# pip uninstall -y gunicorn
```

Install Python3 and pip3, Python's package management tool:

```no-highlight
# apt-get update
# apt-get install -y python3 python3-dev python3-setuptools
# easy_install3 pip
```

Install the Python3 packages required by NetBox:

```no-highlight
# pip3 install -r requirements.txt
```

Replace gunicorn with the Python3 version:

```no-highlight
# pip3 install gunicorn
```

If using LDAP authentication, install the `django-auth-ldap` package:

```no-highlight
# pip3 install django-auth-ldap
```

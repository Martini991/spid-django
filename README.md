SPID Django (former [djangosaml2_spid](https://github.com/peppelinux/djangosaml2_spid))
----------------
![CI build](https://github.com/italia/spid-django/workflows/spid-django/badge.svg)
![Python version](https://img.shields.io/badge/license-Apache%202-blue.svg)
![License](https://img.shields.io/badge/python-3.7%20%7C%203.8%20%7C%203.9-blue.svg)


A SPID Service Provider based on [pysaml2](https://github.com/identitypython/pysaml2).


Introduction
------------
This is a Django application that provides a SAML2 Service Provider
for a Single Sign On with SPID, the Italian Digital Identity System.

Technical documentation on SPID and SAML is available at [Docs Italia](https://docs.italia.it/italia/spid/spid-regole-tecniche/it/34.1.1/index.html)

![big picture](gallery/animated.gif)


Usage
-----

This project comes with a demo Spid button template with both *spid-testenv2* and *spid-saml-check* IDP preconfigured.
You just have to run the example project and put its metadata in spid-testenv2, this way:

````
wget http://localhost:8000/spid/metadata -O conf/sp_metadata.xml
````

then start the example project using `spid-testenv2` or `spid-saml-check` or both, using these environment variables:

````
SPID_SAML_CHECK_REMOTE_METADATA_ACTIVE=True bash run.sh 
SPID_TESTENV2_REMOTE_METADATA_ACTIVE=True bash run.sh 
````


Dependencies
------------

- xmlsec
- python3-dev
- python3-pip
- libssl-dev
- libsasl2-dev


Demo app
------------

Demo application uses **spid-saml-check** and **spid-testenv2** as
SPID IDP, see `example/`.

Prepare environment
````
cd example/
virtualenv -ppython3 env
source env/bin/activate
pip install -r ../requirements.txt
````

Run the example project
 - Your example saml2 configuration is in `spid_config/spid_settings.py`. See djangosaml2 or pysaml2 official docs for clarifications
 - create demo database `./manage.py migrate`
 - run `./manage.py runserver 0.0.0.0:8000` or `SPID_SAML_CHECK_REMOTE_METADATA_ACTIVE=True SPID_TESTENV2_REMOTE_METADATA_ACTIVE=True bash run.sh`
 - run spid-testenv2 and spid-saml-check (docker is suggested)
 - open 'http://localhost:8000'


Demo app (with Docker)
------------

To use Docker compose environment, add to /etc/hosts this line:
````
127.0.0.1	hostnet
````

then use `docker-compose --env-file docker-compose.env up` (the process takes some time) and when the services are up go to http://hostnet:8000/spid/login

**warning**: if you want to change ports of any of the docker-compose services (as, spid-testenv2, spid-saml-check) and/or the FQDN of the docker-compose default network gateway (defaults to `hostnet`) you need to change all the files
under `./example/configs/` to match the new configurations, changing only `./docker-compose.env` will not suffice.


Setup
------------

djangosaml2_spid uses a pySAML2 fork.

* `pip install git+https://github.com/peppelinux/pysaml2.git@pplnx-v6.5.1`
* `pip install git+https://github.com/italia/spid-django`
* Import SAML2 entity configuration in your project settings file: `from spid_config.spid_settings import *`
* Add in `settings.INSTALLED_APPS` the following
  ```
    'djangosaml2',
    'djangosaml2_spid',
    'spid_config'
  ```
  _spid_config_ is your configuration, with statics and templates. See `example` project.
* Add you custom User model, see example project: `AUTH_USER_MODEL = 'custom_accounts.User'`
* Add in `settings.MIDDLEWARE`: `'djangosaml2.middleware.SamlSessionMiddleware'` for [SameSite Cookie](https://github.com/knaperek/djangosaml2#samesite-cookie)
* Add in `settings.AUTHENTICATION_BACKENDS`:
  ```
    'django.contrib.auth.backends.ModelBackend',
    'djangosaml2.backends.Saml2Backend',
  ```
* Generate X.509 certificates and store them to a path, generally in `./certificates`, using [spid-compliant-certificates](https://github.com/italia/spid-compliant-certificates)
* Register the SP metadata to the your test Spid IDP
* Start the django server for tests `./manage.py runserver 0.0.0.0:8000`


Attribute Mapping
-----------------
Is necessary to maps SPID attributes to Django ones.
An example that links fiscalNumber wiht username have been configured in the example project.
This is another example that achieve the same behaviour without changing the default User model.

````
SAML_USE_NAME_ID_AS_USERNAME = False
SAML_DJANGO_USER_MAIN_ATTRIBUTE = 'username'
SAML_CREATE_UNKNOWN_USER = True
SAML_DJANGO_USER_MAIN_ATTRIBUTE_LOOKUP = '__iexact'

SAML_ATTRIBUTE_MAPPING = {
'fiscalNumber': ('username', ),
}
````


Tests
-----

````
pip install requirements-dev.txt
cd example/
coverage erase
coverage run ./manage.py test djangosaml2_spid.tests
coverage report -m
````


Warnings
--------

- The SPID Button template is only for test purpose, please don't use it in production, do your customization instead!
- In a production environment please don't use "remote" as metadata storage, use "local" or "mdq" instead!

Authors
------------

Giuseppe De Marco

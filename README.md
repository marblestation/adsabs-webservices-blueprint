[![Build Status](https://travis-ci.org/adsabs/adsabs-webservices-blueprint.svg?branch=master)](https://travis-ci.org/adsabs/adsabs-webservices-blueprint)

# adsabs-webservices-blueprint

A sample Flask application for backend adsabs (micro) web services. To integrate into the ADS-API, an application must expose a `/resources` route that advertises that application's resources, scopes, and rate limits. 

`GET` on `/resources` should return `JSON` that looks like:

    {
        "/route": {
            "scopes": ["red","green"],
            "rate_limit": [1000,86400],
            "description": "docstring for this route",
            "methods": ["HEAD","OPTIONS","GET"],
        }
    }


To facilitate that, one can define that route explitictly/manually or by using [flask-discoverer](https://github.com/adsabs/flask-discoverer)

## development

A Vagrantfile and puppet manifest are available for development within a virtual machine. To use the vagrant VM defined here you will need to install *Vagrant* and *VirtualBox*. 

  * [Vagrant](https://docs.vagrantup.com)
  * [VirtualBox](https://www.virtualbox.org)

To load and enter the VM: `vagrant up && vagrant ssh`

## database migrations

To make changes to the database associated with the application:

  * Update the database model in `database.py`
  * execute: python sample_application/manage.py db migrate
  * if necessary, make manual changes in the generated migration script (see below)
  * execute: python sample_application/manage.py db upgrade

Notes:

1. Installation of `Flask-Migrate` created the directory `migrations`. Because the sample application uses `SQLALCHEMY_BINDS` to define the database URI, the script `migrations/env.py` needed a manual update: replace `current_app.config.get('SQLALCHEMY_DATABASE_URI')` by `current_app.config.get('SQLALCHEMY_BINDS')['sample']`
2. After running the `migrate` step, some manual editing of the migration script will be necessary because of the definition of one of the table columns: the column `Column(postgresql.ARRAY(String))` gets translated into `sa.Column('bar', postgresql.ARRAY(String()), nullable=True)` in the migration, which will throw an exception if left unedited. The entry `String()` has to be changed into `sa.String()`. This is true in general for all types used in `ARRAY`.
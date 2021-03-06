# Connectors integration tests

### Structure

* **config/** configuration files used to generate installation tests for every connector
* **expected/** test results expectations
* **fixtures/** fixtures for every tested connector
* **results/** test results
* **sql/** all the tests we have for all the tested connectors
* **template/** folder with the templates used to generate installation tests
* **test-generator.sh** script used to generate installation tests using templates and config files.

### Type of tests

#####Installation tests

Autogenerated tests that are built by using a template, which must exist in the `template` folder, and a configuration file, that must exist in the config folder.

The template name must follow the pattern: `connector_name_installation_test.tpl`. For example: `mysql_installation_test.tpl`. The configuration file should be a copy of `connector.sample.config` and be named following the pattern `connector_name.config`. For example: `mysql.config`

Inside the template you should include all the needed commands to create the connection with the connector through FDW:

- Server creation with CREATE SERVER
- User mapping creation using CREATE USER MAPPING
- Import foreign schema to bring a complete schema (if the host has schemas), only some tables, or create a table from a query.

#####Query tests:

This kind of tests is not automatically generated. The objective is to test if it's possible to execute queries to the multiple tested connectors and check that the return value is correct.

Other objective of these tests is to check the viability of use for certain data types as they become available.

### How to add tests for a new connector

In order to add new tests for a new connector, you've to follow the following steps:

- Add the needed files to be able to autogenerate tests and expectations to tests installation for the new connector:
    - Create a new configuration file inside the `configuration` folder following the pattern for its name: `connector_name.config`.
    - **Config file must not be stored in the repository, we only [store encrypted config files](#configuration_files)**
    - After that, you need to create a new template inside the `templates` folder following the pattern for its name: `connector_name_*.tpl`. Right now were are following this convention: `connector_name_installation_test.tpl`
- Add a new fixture file with prepared data for the new connector. For example: `fixtures\mysql_fixtures.sql`
- [optional] Add command to load the new fixture in the `load_fixtures.sh` script. This script is used by Travis to load the fixtures in the test databases.
- Create tests and expectations to test queries with the new connector. For example: `sql\mysql_20_query_test.sql`

### Configuration files

Configuration files contain sensitive information that must not be stored in the repository. In this folder we only store encrypted files `.config.enc` that are going to be used by Travis.

When we want to change or add a new configuration file, we have to encrypt the file to be used by Travis. Before all you should read [this post](https://docs.travis-ci.com/user/encrypting-files/) to understand how it works and how to setup the environment to encrypt the files.

When you use the command `travis encrypt-file` you should provide a key and a IV value. The key needs to be 64 character long and IV needs to be 32 character long. And both of them must be a valid hex number.

I've been using these ruby commands to generate them:

```
# Key
ruby -rsecurerandom -e 'puts SecureRandom.hex(32).chomp'
# IV
ruby -rsecurerandom -e 'puts SecureRandom.hex(16).chomp'
```

An example would be:

```
travis encrypt-file -K "fookey" --iv "foodiv" example.config
```

After encrypting the file, commit only the `.config.enc` file and add the `openssl` command execution to the `.travis.yml` file.
### How to execute the tests

To launch the tests you have to execute the command `make integration_tests` after installing the extension by using `make install` command

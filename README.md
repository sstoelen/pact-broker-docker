Dockerised Pact Broker [![Build Status](https://travis-ci.org/pact-foundation/pact-broker-docker.svg)](https://travis-ci.org/pact-foundation/pact-broker-docker)
==================

This repository deploys [Pact Broker][pact-broker] using lightweight containers using Docker. You can pull the pactfoundation/pact-broker image from [Dockerhub][pact-broker-docker].

## Prerequisites

* A running postgresql database and the ability to connect to it (see [POSTGRESQL.md][postgres]).
* If on Mac, you will need the `timeout` or `gtimeout` function. You can install `gtimeout` using `brew install coreutils`.

## Getting Started

1. [Install Docker][docker]
2. Prepare your environment if you are not running postgresql in a docker container. Setup the pact broker connection to the database through the use of the following environment variables. If you want to use a disposable postgres docker container just do `export DISPOSABLE_PSQL=true` before running the [script/test.sh][test-script].

For a postgres or mysql database:

    * PACT_BROKER_DATABASE_ADAPTER (optional, defaults to 'postgres', see note below.)
    * PACT_BROKER_DATABASE_USERNAME
    * PACT_BROKER_DATABASE_PASSWORD
    * PACT_BROKER_DATABASE_HOST
    * PACT_BROKER_DATABASE_PORT (optional, defaults to the default port for the specified adapter)
    * PACT_BROKER_DATABASE_NAME

Adapter can be 'postgres' (recommended) or 'mysql2' (please note that future JSON search features may not be supported on mysql).

For an sqlite database (only recommended for investigation/spikes, as it will be disposed of with the container unless you mount it from an external file system):

  * PACT_BROKER_DATABASE_ADAPTER (set to 'sqlite')
  * PACT_BROKER_DATABASE_NAME (arbitrary name eg. pact_broker.sqlite)

3. Test the pact broker environment by executing [script/test.sh][test-script]

## Notes

* On OSX, if you are not using Docker native, use `docker-machine ip $(docker-machine active)` to get the IP of the VirtualBox, and connect on port 9292.
* The application makes use of the Puma application server.
* Apart from creating a database no further preparation is required.

## Using basic auth

To enable basic auth, run your container with:

* `PACT_BROKER_BASIC_AUTH_USERNAME`
* `PACT_BROKER_BASIC_AUTH_PASSWORD`
* `PACT_BROKER_BASIC_AUTH_READ_ONLY_USERNAME`
* `PACT_BROKER_BASIC_AUTH_READ_ONLY_PASSWORD`

Developers should use the read only credentials on their local machines, and the CI should use the read/write credentials. This will ensure that pacts and verification results are only published from your CI.

Note that the [verification status badges][badges] are not protected by basic auth, so that you may embed them in README markdown.

## Heartbeat URL

If you are using the docker container within an AWS autoscaling group, and you need to make a heartbeat URL publicly available, set `PACT_BROKER_PUBLIC_HEARTBEAT=true`. No database connection will be made during the execution of this endpoint.

## Using SSL

See the [Pact Broker configuration documentation][reverse-proxy].

## Setting the log level

Set the environment variable `PACT_BROKER_LOG_LEVEL` to one of `DEBUG`, `INFO`, `WARN`, `ERROR`, or `FATAL`.

## Webhook whitelists

* PACT_BROKER_WEBHOOK_HOST_WHITELIST - a space delimited list of hosts (eg. `github.com`), network ranges (eg. `10.2.3.41/24`, or regular expressions (eg. `/.*\\.foo\\.com$/`). Regular expressions should start and end with a `/` to differentiate them from Strings. Note that backslashes need to be escaped with a second backslash. Please read the [Webhook whitelists][webhook-whitelist] section of the Pact Broker configuration documentation to understand how the whitelist is used. Remember to use quotes around this value as it may have spaces in it.
* PACT_BROKER_WEBHOOK_SCHEME_WHITELIST - a space delimited list (eg. `http https`). Defaults to `https`.

## Other environment variables

* PACT_BROKER_BASE_EQUALITY_ONLY_ON_CONTENT_THAT_AFFECTS_VERIFICATION_RESULTS - `true` by default, may be set to `false`.
* PACT_BROKER_ORDER_VERSIONS_BY_DATE - `true` by default, may be set to `false`.
* PACT_BROKER_DISABLE_SSL_VERIFICATION - `false` by default, may be set to `true`.

## General Pact Broker configuration and usage

Documentation for the Pact Broker application itself can be found in the Pact Broker [wiki][pact-broker-wiki].

## Running with Docker Compose

For a quick start with the Pact Broker and Postgres, we have an example
[Docker Compose][docker-compose] setup you can use:

1. Modify the `docker-compose.yml` file as required.
2. Run `docker-compose build` to build the pact_broker container locally.
3. Run `docker-compose up` to get a running Pact Broker and a clean Postgres database.

Now you can access your local broker:

```sh
curl -v http://localhost # you can visit in your browser too!

# SSL endpoint, note that URLs in response contain https:// protocol
curl -v -k https://localhost:8443
```

_NOTE: this image should be modified before using in Production, in particular, the use of hard-coded credentials_

## Running with Openshift

See [pact-broker-openshift][pact-broker-openshift] for an example config file.

# Troubleshooting

See the [Troubleshooting][troubleshooting] page on the wiki.

[docker]: https://docs.docker.com/install/
[pact-broker]: https://github.com/pact-foundation/pact_broker
[pact-broker-docker]: https://hub.docker.com/r/pactfoundation/pact-broker/
[pact-broker-openshift]: https://github.com/jaimeniswonger/pact-broker-openshift
[badges]: https://github.com/pact-foundation/pact_broker/wiki/Provider-verification-badges
[troubleshooting]: https://github.com/pact-foundation/pact-broker-docker/wiki/Troubleshooting
[postgres]: https://github.com/pact-foundation/pact-broker-docker/blob/master/POSTGRESQL.md
[test-script]: https://github.com/pact-foundation/pact-broker-docker/blob/master/script/test.sh
[docker-compose]: https://github.com/pact-foundation/pact-broker-docker/blob/master/docker-compose.yml
[pact-broker-wiki]: https://github.com/pact-foundation/pact_broker/wiki
[reverse-proxy]: https://github.com/pact-foundation/pact_broker/wiki/Configuration#running-the-broker-behind-a-reverse-proxy
[webhook-whitelist]: https://github.com/pact-foundation/pact_broker/wiki/Configuration#webhook-whitelists

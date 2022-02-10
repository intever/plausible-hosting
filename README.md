# Self Hosting [Plausible](https://plausible.io/docs/self-hosting) on [Fly.io](https://fly.io/)

This repo is based on the [docker-compose version](https://github.com/plausible/hosting) published by Plausible

## Overview

There are 4-5 Services that are required to run plausible: The plausible service itself, a postgres DB, a Clickhouse Server, an SMTP Service, and an (optional) geolocation database. The self-hosting guide and the associated repository are a great place to start, but they only provide a docker-compose file and kubernetes examples. To run on fly.io, I'd like to deploy each service separately.

## Postgres

Refer to the [Fly.io documenation](https://fly.io/docs/reference/postgres/) for complete instructions.

```
flyctl postgres create
? App Name: plausible-db
? Select organization: <YOUR ORGANIZATION>
? Select region: ewr (Secaucus, NJ (US))
? Select configuration: Development - Single node, 1x shared CPU, 256MB RAM, 1GB disk
```

Save the credentials somewhere safe

## Clickhouse

(from the clickhouse directory)

```
flyctl volumes create plausible_clickhouse_data --region ewr --size 1
flyctl deploy
```

## Plausible

(from the plausible directory)

```
flyctl postgres attach --postgres-app plausible-db

openssl rand -base64 64 | tr -d '\n' ; echo

flyctl secrets set SECRET_KEY_BASE=<FROM ABOVE>

<SET ADDITIONAL CONFIGURATION (See Below)>

flyctl deploy
```

The default VM was running out of memory:

```
fly scale memory 1024
```

```
fly ssh console

entrypoint.sh db init-admin
```

### [DNS Entries / SSL](https://fly.io/docs/app-guides/custom-domains-with-fly/)
A Record
AAAA Record

```
flyctl certs create <BASE_URL>
```

### Secrets & Configuration

Please refer the [configuration documentation](https://plausible.io/docs/self-hosting-configuration#mailersmtp-setup)

These are the configurations we used:

```
ADMIN_USER_NAME
ADMIN_USER_EMAIL
ADMIN_USER_PWD

BASE_URL
SECRET_KEY_BASE
DISABLE_REGISTRATION=true

MAILER_ADAPTER=Bamboo.PostmarkAdapter
POSTMARK_API_KEY
MAILER_EMAIL

GOOGLE_CLIENT_ID
GOOGLE_CLIENT_SECRET
```

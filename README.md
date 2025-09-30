# ns8-matrix

A NethServer 8 module for [Matrix](https://matrix.org/) chat service integration with LDAP authentication via OpenID Connect.

This module provides a complete Matrix chat solution including:
- **Dex** (OpenID Connect Provider) - Connects to NethServer LDAP for user authentication
- **Synapse** (Matrix Homeserver) - Core Matrix server with OIDC authentication
- **Element Web** (Web Client) - Web interface for Matrix chat

## Features

- Seamless integration with NethServer 8 LDAP user directory
- OpenID Connect authentication through Dex
- Configurable domain names for both Matrix server and Element web client
- Automatic SSL/TLS certificate management via Traefik
- Matrix Federation support
- Persistent data storage

## Install

Instantiate the module with:

    add-module ghcr.io/nethserver/matrix:latest 1

The output of the command will return the instance name.
Output example:

    {"module_id": "matrix1", "image_name": "matrix", "image_url": "ghcr.io/nethserver/matrix:latest"}

## Configure

Let's assume that the matrix instance is named `matrix1`.

Launch `configure-module`, by setting the following parameters:
- `synapse_domain_name`: The fully qualified domain name for the Matrix server (e.g., `matrix.example.com`)
- `element_domain_name`: The fully qualified domain name for the Element web client (e.g., `chat.example.com`)

Example:

    api-cli run module/matrix1/configure-module --data '{"synapse_domain_name": "matrix.example.com", "element_domain_name": "chat.example.com"}'

The above command will:
- Configure Dex as OpenID Connect provider with LDAP backend
- Start and configure the Synapse Matrix homeserver with OIDC authentication
- Deploy Element Web client configured to connect to the local Synapse instance
- Set up Traefik routes for both domains with automatic SSL certificates

Access your Matrix installation:
- Matrix server: `https://matrix.example.com`
- Element web client: `https://chat.example.com`

## LDAP Integration

The module automatically discovers LDAP settings from the NethServer 8 configuration. 
Dex is configured to connect to the NS8 LDAP proxy to authenticate users against 
the centralized user directory.

The LDAP discovery happens every time the services start through the `bin/discover-ldap` 
script, which refreshes the `state/ldap.env` file with current LDAP configuration.

If LDAP settings change while the module is running, the event handler 
`events/ldap-changed/10reload_services` will restart the affected services.

## Matrix Federation

The module supports Matrix federation by configuring the necessary server delegation 
and federation settings. The Synapse server is configured with the proper server name
and federation endpoints are exposed through Traefik.

## Uninstall

To uninstall the instance:

    remove-module --no-preserve matrix1

## Testing

Test the module using the `test-module.sh` script:


    ./test-module.sh <NODE_ADDR> ghcr.io/nethserver/matrix:latest

The tests are made using [Robot Framework](https://robotframework.org/)

## UI translation

Translated with [Weblate](https://hosted.weblate.org/projects/ns8/).

To setup the translation process:

- add [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repository
- add your repository to [hosted.weblate.org]((https://hosted.weblate.org) or ask a NethServer developer to add it to ns8 Weblate project

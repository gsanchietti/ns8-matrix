# ns8-matrix

:warning: This project is just a proof of concept to evaluate Matrix integration with NethServer 8.
The goal is to provide a chat engine for NethVoice and NethCTI modules in the future: see https://github.com/NethServer/dev/issues/7648.
This module is in early development and should be used for testing purposes only. Do not use in production environments. :warning:

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
- `lets_encrypt`: Set to `true` to enable automatic SSL certificate generation via Let's Encrypt for both domains.
- `dex_ldap_domain`: The LDAP domain name to be used by Dex for authentication. If not provided, the default LDAP domain will be used.

Example:

    api-cli run module/matrix1/configure-module --data '{"synapse_domain_name": "matrix.example.com", "element_domain_name": "chat.example.com", "lets_encrypt": true, "dex_ldap_domain": "users.example.com"}'

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

## Next steps and potential improvements

Stack explanation:
- [Synapse](https://element-hq.github.io/synapse/latest/setup/installation.html) is the standard de-fact Matrix homeserver implementation
- Dex is used as OpenID Connect Provider to authenticate users against LDAP, it's not strictly required unless MAS (Matrix Authentication Service) is used (currently MAS is not implemented in this module)
- Element web is the official Matrix web client, it can be replaced with other web client clients like [Cinny](https://cinny.in/)

Alternative stacks:
- Remove Dex, use old LDAP integration [matrix-synapse-ldap3](https://github.com/matrix-org/matrix-synapse-ldap3) (currently deprecated
)
- Use [MAS](https://element-hq.github.io/matrix-authentication-service/) to add more authentication methods (SAML, OIDC, LDAP, etc.) and integrate with [authentik](https://goauthentik.io/) [1](https://integrations.goauthentik.io/chat-communication-collaboration/matrix-synapse/) [2](https://element-hq.github.io/matrix-authentication-service/setup/sso.html#authentik)

### To be done

If the current stack will be used, the following must be made.

Element web:
- do not display the registration option
- force [SSO login](https://github.com/element-hq/element-web/blob/develop/docs/config.md#sso-setup)
- disable matrix.org as default server, set the local Synapse server as default
- customize the UI with branding (logo, colors, etc.)

Dex:
- implement secure secret keys, do not use hardcoded ones like current `synapse-secret`
- customize the UI with branding (logo, colors, etc.)
- export preferred user name from LDAP to synapse (currently Dex exports only email and username)
- add support for LDAP groups (currently Dex exports only users)
- add support for Active Directory
- switch from Sqlite to Postgres

Synapse:
- switch from Sqlite to Postgres 

NethServer module:
- UI: allow to configure the connected User Domain
- UI: add Let's Encrypt option

NethVoice CTI integration:
- test current available client integrations
  - [Chatterbox](https://github.com/element-hq/chatterbox)
  - [Cactus](https://cactus.chat/)
  - Plain [Javascript SDK](https://github.com/matrix-org/matrix-js-sdk)
- implement authentication, there 2 possible ways:
  - add support for OIDC ID inside [CTI middleware](https://github.com/nethesis/nethcti-middleware): the user authenticate against the Oauth 2 provider (like Dex or authentik, if 2FA is required) and the middleware uses the ID token to authenticate against Synapse with the obtained token
  - use normal LDAP auhtentication: the CTI middleware takes care also to authenticate against Synapse using LDAP credentials

### Future improvements

- Evaluate if [Sygnal Push Gateway](https://github.com/element-hq/sygnal) is required (it should not)
- Add support for [VoIP and Video calls](https://element-hq.github.io/synapse/latest/usage/calls.html) with [Element Call](https://github.com/element-hq/element-call)
- Add Whatsapp Bridge with [mautrix-whatsapp](https://github.com/mautrix/whatsapp)
- Add Telegram Bridge with [mautrix-telegram](https://github.com/mautrix/telegram)

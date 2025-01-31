[![Bonsai Asset Badge](https://img.shields.io/badge/Sensu%20Teams%20Handler-Download%20Me-brightgreen.svg?colorB=89C967&logo=sensu)](https://bonsai.sensu.io/assets/mildgrim/sensu-teams-handler) [![Build Status](https://app.travis-ci.com/mildgrim/sensu-teams-handler.svg?branch=main)](https://app.travis-ci.com/mildgrim/sensu-teams-handler)

# Sensu Teams Handler

- [Overview](#overview)
- [Usage examples](#usage-examples)
  - [Help output](#help-output)
  - [Environment variables](#environment-variables)
  - [Templates](#templates)
  - [Annotations](#annotations)
- [Configuration](#configuration)
  - [Asset registration](#asset-registration)
  - [Handler definition](#handler-definition)
  - [Check definition](#check-definition)
- [Installation from source and contributing](#installation-from-source-and-contributing)

## Overview


The [Sensu Teams Handler][0] is a [Sensu Event Handler][3] that sends event data
to a configured Teams channel. It is a rewrite by [Sebastian Mildgrim](https://github.com/mildgrim) of the Sensu Inc written [sensu-slack-handler](https://github.com/sensu/sensu-slack-handler)

## Usage examples

### Help output

Help:

```
Usage:
  sensu-teams-handler [flags]
  sensu-teams-handler [command]

Available Commands:
  help        Help about any command
  version     Print the version number of this plugin

Flags:
  -s, --sender string                 The name that messages will be sent as (default "Sensu")
  -d, --description-template string   The Teams notification output template, in Golang text/template format (default "{{ .Check.Output }}")
  -h, --help                          help for sensu-teams-handler
  -u, --sensu-url string              A URL to the Sensu dashboard (default "http://localhost:3000")
  -t, --is-test string                Specify if this is a test run (default "false")
  -w, --webhook-url string            The webhook url to send messages to
```

### Environment variables

|Argument               |Environment Variable       |
|-----------------------|---------------------------|
|--webhook-url          |TEAMS_WEBHOOK_URL          |
|--sender               |TEAMS_SENDER               |
|--is-test              |TEAMS_IS_TEST              |
|--sensu-url            |TEAMS_SENSU_URL            |
|--description-template |TEAMS_DESCRIPTION_TEMPLATE |


**Security Note:** Care should be taken to not expose the webhook URL for this handler by specifying it
on the command line or by directly setting the environment variable in the handler definition.  It is
suggested to make use of [secrets management][7] to surface it as an environment variable.  The
handler definition above references it as a secret.  Below is an example secrets definition that make
use of the built-in [env secrets provider][8].

```yml
---
type: Secret
api_version: secrets/v1
metadata:
  name: teams-webhook-url
spec:
  provider: env
  id: TEAMS_WEBHOOK_URL
```

### Templates

This handler provides options for using templates to populate the values
provided by the event in the message sent via Teams. More information on
template syntax and format can be found in [the documentation][9]


### Annotations

All arguments for this handler are tunable on a per entity or check basis based
on annotations. The annotations keyspace for this handler is
`sensu.io/plugins/teams/config`.

**NOTE**: Due to [check token substituion][10], supplying a template value such
as for `description-template` as a check annotation requires that you place the
desired template as a [golang string literal][11] (enlcosed in backticks)
within another template definition.  This does not apply to entity annotations.

#### Examples

To customize the sender for a given entity, you could use the following
sensu-agent configuration snippet:

```yml
# /etc/sensu/agent.yml example
annotations:
  sensu.io/plugins/teams/config/sender: 'server1'
```

## Configuration

### Asset registration

Assets are the best way to make use of this handler. If you're not using an asset, please consider doing so! If you're using sensuctl 5.13 or later, you can use the following command to add the asset:

`sensuctl asset add mildgrim/sensu-teams-handler`

If you're using an earlier version of sensuctl, you can download the asset
definition from [this project's Bonsai Asset Index
page][6].

### Handler definition

Create the handler using the following handler definition:

```yml
---
api_version: core/v2
type: Handler
metadata:
  namespace: default
  name: teams
spec:
  type: pipe
  command: sensu-teams-handler --sensuUrl 'http://localhost:3000' --sender 'Sensu'
  filters:
  - is_incident
  runtime_assets:
  - sensu/sensu-teams-handler
  secrets:
  - name: TEAMS_WEBHOOK_URL
    secret: teams-webhook-url
  timeout: 10
```
**Note**: The library used in the Sensu SDK for this plugin requires that if your Teams webhook URL is listed as an environment variable, the URL cannot be surrounded by quotes. 

**Security Note**: The Teams webhook URL should always be treated as a security
sensitive configuration option and in this example, it is loaded into the
handler configuration as an environment variable using a [secret][5]. Command
arguments are commonly readable from the process table by other unprivaledged
users on a system (ex: ps and top commands), so it's a better practise to read
in sensitive information via environment variables or configuration files on
disk. The --webhook-url flag is provided as an override for testing purposes.

### Check definition

```
api_version: core/v2
type: CheckConfig
metadata:
  namespace: default
  name: dummy-app-healthz
spec:
  command: check-http -u http://localhost:8080/healthz
  subscriptions:
  - dummy
  publish: true
  interval: 10
  handlers:
  - teams
```

### Proxy Support

This handler supports the use of the environment variables HTTP_PROXY,
HTTPS_PROXY, and NO_PROXY (or the lowercase versions thereof). HTTPS_PROXY takes
precedence over HTTP_PROXY for https requests.  The environment values may be
either a complete URL or a "host[:port]", in which case the "http" scheme is assumed.

## Installing from source and contributing

Download the latest version of the sensu-teams-handler from [releases][4],
or create an executable from this source.

### Compiling

From the local path of the sensu-teams-handler repository:
```
go build
```

To contribute to this plugin, see [CONTRIBUTING](https://github.com/sensu/sensu-go/blob/master/CONTRIBUTING.md)

[0]: https://github.com/mildgrim/sensu-teams-handler
[1]: https://github.com/sensu/sensu-go
[3]: https://docs.sensu.io/sensu-go/latest/reference/handlers/#how-do-sensu-handlers-work
[4]: https://github.com/sensu/sensu-teams-handler/releases
[5]: https://docs.sensu.io/sensu-go/latest/reference/secrets/
[6]: https://bonsai.sensu.io/assets/mildgrim/sensu-teams-handler
[7]: https://docs.sensu.io/sensu-go/latest/guides/secrets-management/
[8]: https://docs.sensu.io/sensu-go/latest/guides/secrets-management/#use-env-for-secrets-management
[9]: https://docs.sensu.io/sensu-go/latest/observability-pipeline/observe-process/handler-templates/
[10]: https://docs.sensu.io/sensu-go/latest/observability-pipeline/observe-schedule/checks/#check-token-substitution
[11]: https://golang.org/ref/spec#String_literals

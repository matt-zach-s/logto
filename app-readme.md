# {{ .nuon.app.name }}

Logto is an open-source identity platform: OIDC, OAuth 2.1 and SAML sign-in, with an admin console.
This install runs it in your own AWS account on EKS, with an in-cluster Postgres database.

Install: `{{ .nuon.name }}` (`{{ .nuon.id }}`)

## Endpoints

{{ $auth := "auth" }}{{ $admin := "admin" }}{{ with .nuon.inputs }}{{ with .inputs }}{{ with .auth_subdomain }}{{ $auth = . }}{{ end }}{{ with .admin_subdomain }}{{ $admin = . }}{{ end }}{{ end }}{{ end }}
{{ with .nuon.sandbox }}{{ with .outputs }}{{ with .nuon_dns }}{{ with .public_domain }}{{ with .name }}
| Service | URL |
| ------- | --- |
| Sign-in and OIDC endpoint | [https://{{ $auth }}.{{ . }}](https://{{ $auth }}.{{ . }}) |
| Admin console | [https://{{ $admin }}.{{ . }}](https://{{ $admin }}.{{ . }}) |

Open the admin console first to create the admin account.
{{ else }}The public domain appears here once the sandbox is provisioned.{{ end }}{{ else }}The public domain appears here once the sandbox is provisioned.{{ end }}{{ else }}DNS details appear here once the sandbox is provisioned.{{ end }}{{ else }}Sandbox outputs appear here once the sandbox is provisioned.{{ end }}{{ else }}The sandbox has not been provisioned yet.{{ end }}

## Components

{{ with .nuon.components }}
| Component | Status |
| --------- | ------ |
{{- range $name, $component := . }}
| `{{ $name }}` | {{ with $component }}{{ with .status }}`{{ . }}`{{ else }}pending{{ end }}{{ else }}pending{{ end }} |
{{- end }}
{{ else }}
No components have been deployed yet.
{{ end }}

## What runs where

- `logto` (namespace `logto`): Logto `ghcr.io/logto-io/logto`. Port 3001 serves sign-in and OIDC, port 3002 the admin console.
  An init container seeds the database on first boot and does nothing on later boots.
- `postgres` (namespace `logto`): Postgres 17 on a 20 GiB encrypted gp3 volume. The password is a Nuon
  secret, auto-generated unless you set one, and synced to the Kubernetes secret `logto-postgres`.
- `ingress`: one internet-facing ALB, with TLS from the `certificate` component, serving both hostnames.

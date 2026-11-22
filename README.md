# openBIS service for Kubernetes on Wodby

Run openBIS, the lab notebook and inventory manager, as a reusable Kubernetes application service with Wodby.

This repository defines the Wodby service manifest and operational configuration for openBIS.

- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Wodby stacks using this service

- [openBIS application stack](https://github.com/wodby/stack-openbis)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `openbis` |
| Type | Application service |
| Versions | `7.0` |
| Workloads | `main` (StatefulSet, primary) |
| Containers | `openbis` using `wodby/openbis` |
| Endpoints | `openbis`: HTTP 8080 (main), 8081 and 8085, all private |
| Service links | PostgreSQL (`postgres`), required; SMTP server (`sendmail`), optional |
| Application build | Not buildable from application source |
| Helm | chart `oci://registry-1.docker.io/wodby/stateful`; version `0.4.0` |
| Configuration and operations | 1 setting, 1 volume |

## Use this service

Use this service through the [openBIS application stack](https://github.com/wodby/stack-openbis), or reference `openbis`
from a custom Wodby stack together with the
[openBIS Nginx service](https://github.com/wodby/service-openbis-nginx), which is its public entry point.

A service is a reusable component and does not deploy by itself. The stack defines its links, settings, versions,
resources, and relationship to the rest of the application.

## Database

openBIS needs PostgreSQL with three databases and prepared transactions turned on. The PostgreSQL link asks Wodby for
the two extra databases next to the main one, and the openBIS stack turns on prepared transactions on its PostgreSQL
service.

## Upgrades

openBIS migrates its database schemas when a newer version starts. Back up the databases and the `data` volume first;
moving back to an older version is not supported.

## Maintain a custom version

1. Fork this repository.
2. Edit the service manifest and referenced files.
3. Import the repository as a [Git-backed service](https://wodby.com/docs/2.0/services/create/#create-a-git-backed-service).
4. Reference the service from a stack manifest.

Keep service, workload, container, endpoint, link, volume, config, and derivative names stable unless dependent stacks
and app-level overrides are updated at the same time.

Validate the manifest with:

```bash
wodby service validate-manifest service.yml --org <org-id> --include GUIDANCE.md
```

See the [service manifest reference](https://wodby.com/docs/2.0/services/template/) and the
[managed services index](https://github.com/wodby/services).

# openBIS on Wodby

What Wodby sets up for openBIS on this service. openBIS runs from the `wodby/openbis` image: the application server
with the lab notebook (port 8080), the data store server (8081) and the file server (8085), in one container, as one
replica, as the non-root user `openbis` (uid 1001). The ports are private; the openBIS Nginx service routes
`/openbis`, `/datastore_server` and `/afs-server` to them and sends `/` to the lab notebook.

## Linked services

| Link | Variables |
| --- | --- |
| PostgreSQL (`postgres`, required) | `OPENBIS_DB_HOST`, `OPENBIS_DB_PORT`, `OPENBIS_DB_NAME`, `OPENBIS_DB_MESSAGES_NAME`, `OPENBIS_DB_PATHINFO_NAME`, `OPENBIS_DB_USER`, `OPENBIS_DB_PASSWORD` |
| SMTP server (`sendmail`, optional) | `SMTP_HOST`, `SMTP_PORT` |

openBIS keeps its data in three PostgreSQL databases. The link asks Wodby for two extra databases next to the main
one, `<main database>_messages` and `<main database>_pathinfo`; Wodby creates them on the same server when openBIS is
deployed and gives the same user access. On the first start the image prepares each empty database so openBIS creates
its schema there; no superuser rights are needed.

The PostgreSQL server must allow prepared transactions: openBIS commits changes that span its application server and
file server with two-phase commit. The openBIS stack sets **Max prepared transactions** on its PostgreSQL service to
64. Without it, openBIS operations that span both servers fail when they commit.

The link selects the PostgreSQL 18 service. Managed PostgreSQL servers are not supported, because openBIS connects
without TLS.

## What the manifest sets

- The admin password: the Admin password setting is applied to the openBIS user `admin` on every start, so changing
  the setting and deploying changes the password. Other users are added in openBIS.
- Public addresses: openBIS builds its links and the file server address from the environment's primary host
  (`WODBY_PRIMARY_HOST`), over HTTPS.
- Email: with the SMTP link, openBIS sends email through the linked server from `noreply@` the primary host.
- Memory: the Java heaps of the three servers take 45 %, 12 % and 12 % of the container's memory limit
  (`OPENBIS_AS_MAX_RAM_PERCENTAGE`, `OPENBIS_DSS_MAX_RAM_PERCENTAGE`, `OPENBIS_AFS_MAX_RAM_PERCENTAGE`). Raise the limit
  rather than the percentages: the rest is used by the servers outside their heaps.

## Data

Everything openBIS keeps outside the databases is on the `data` volume at `/data`: stored files and data sets
(`/data/openbis`), configuration (`/data/etc`) and keys generated on the first start (`/data/.wodby`). Wodby's database
backups and imports cover only the main database; the volume and the messages and path info databases are not
backed up by Wodby.

## Changing configuration

The configuration of the three servers is rendered on every start. To add your own openBIS properties, such as another
authentication service, put them into `/data/etc/as/service.properties.local` (application server),
`/data/etc/dss/service.properties.local` or `/data/etc/afs/service.properties.local` and restart the service; they are
appended, and later lines win.

## Upgrades

openBIS migrates its database schemas when a newer version starts. Back up the databases and the `data` volume first;
moving back to an older version is not supported.

## Check the result

From inside the environment, `/openbis/webapp/eln-lims/version.txt` on port 8080 answers once the application server
runs; the service is probed with it. The container stops, and Kubernetes restarts it, when one of the three servers
stops. The first start creates the schemas and the lab notebook's master data, which takes a few minutes.

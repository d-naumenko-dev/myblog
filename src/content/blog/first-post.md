---
title: "Upgrading PostgreSQL from version 11 to 12"
description: "How to upgrade PostgreSQL from 11 to 12 using pg_upgrade."
pubDate: "Nov 28 2019"
heroImage: "/blog-placeholder-3.jpg"
---

PostgreSQL 12 was released on 2019-10-03. You can upgrade from an older version either with `pg_dumpall` or with `pg_upgrade`. The variant described below uses `pg_upgrade`.

## Install PostgreSQL 12

```bash
sudo apt-get update
sudo apt-get install postgresql-12 postgresql-server-dev-12
```

Move your custom settings from the old configs into the new ones. It's convenient to review the differences between the configs of the two versions with:

```bash
diff /etc/postgresql/11/main/postgresql.conf /etc/postgresql/12/main/postgresql.conf
diff /etc/postgresql/11/main/pg_hba.conf /etc/postgresql/12/main/pg_hba.conf
```

## Stop the running PostgreSQL

```bash
sudo systemctl stop postgresql.service
```

Switch to the directory for temporary files. Logs will be written there and some scripts will be added:

```bash
cd /tmp
```

Start working on the command line as the `postgres` user:

```bash
sudo su postgres
```

## Check the clusters

Safely check the clusters, without modifying any data:

```bash
/usr/lib/postgresql/12/bin/pg_upgrade \
  --old-datadir=/var/lib/postgresql/11/main \
  --new-datadir=/var/lib/postgresql/12/main \
  --old-bindir=/usr/lib/postgresql/11/bin \
  --new-bindir=/usr/lib/postgresql/12/bin \
  --old-options '-c config_file=/etc/postgresql/11/main/postgresql.conf' \
  --new-options '-c config_file=/etc/postgresql/12/main/postgresql.conf' \
  --check
```

## Migrate the data

If there are no errors, perform the data migration (if you don't need to copy the files into the new cluster, use the `--link` parameter — hard links to the old cluster will be used, without copying):

```bash
/usr/lib/postgresql/12/bin/pg_upgrade \
  --old-datadir=/var/lib/postgresql/11/main \
  --new-datadir=/var/lib/postgresql/12/main \
  --old-bindir=/usr/lib/postgresql/11/bin \
  --new-bindir=/usr/lib/postgresql/12/bin \
  --old-options '-c config_file=/etc/postgresql/11/main/postgresql.conf' \
  --new-options '-c config_file=/etc/postgresql/12/main/postgresql.conf'
```

Return to your regular user:

```bash
exit
```

## Swap the ports

Your old PostgreSQL most likely used port 5432, while the new one uses port 5433 by default. Swap them.

```bash
sudo vim /etc/postgresql/12/main/postgresql.conf
# change "port = 5433" to "port = 5432"

sudo vim /etc/postgresql/11/main/postgresql.conf
# change "port = 5432" to "port = 5433"
```

## Start PostgreSQL

```bash
sudo systemctl start postgresql.service
```

Work as the `postgres` user:

```bash
sudo su postgres
```

Check the version of the running PostgreSQL:

```bash
psql -c "SELECT version();"
```

The new cluster has no statistics yet. You need to run `ANALYZE` over the cluster. For this, `pg_upgrade` created the script `analyze_new_cluster.sh`. Run it.

```bash
./analyze_new_cluster.sh
```

Return to your regular user:

```bash
exit
```

## Remove the old version

See which old PostgreSQL versions remain in the system.

```bash
apt list --installed | grep postgresql
```

Remove the old PostgreSQL versions, for example:

```bash
sudo apt-get remove postgresql-11
```

Remove the old configuration:

```bash
sudo rm -rf /etc/postgresql/11/
```

Log in as the `postgres` user one last time:

```bash
sudo su postgres
```

Delete the old cluster's data:

```bash
./delete_old_cluster.sh
```

The upgrade is complete!

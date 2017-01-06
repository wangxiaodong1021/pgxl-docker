# PGXL-docker with PostGis extension

[Postgre-XL](http://www.postgres-xl.org/) is a new MPP fork of [PostgreSQL](http://www.postgresql.org).

This version is a test of Postgre-XL with PostGis extension with docker, the default configuration file runs several data nodes, 1 gtm and 1 coordinator.
The docker configuration is not fully functional and is unsecure (run with docker privilegied mode, no password for postgresql coordinator).

## Tutorial

You need python and docker and two python package docker-py and fig.
```
$ pip install fig
```
### 1. Building
```
$ git clone https://github.com/postmind-net/pgxl-docker.git
```
Building the container:
```
$ fig build
```
This script generate ssh key in the directory pom/.ssh to allow container passwordless ssh connections.
```
$ fig run pgxl key
```
### 2. Starting containers

We use fig to facilitate container building and running. The fig scale command create several pgxl containers. Here 4 containers are started.
```
$ fig scale pgxl=4
Starting pgxldocker_pgxl_1...
Starting pgxldocker_pgxl_2...
Starting pgxldocker_pgxl_3...
Starting pgxldocker_pgxl_4...
```
You can check the containers status with fig ps:
```
$ fig ps

      Name                     Command             State     Ports
------------------------------------------------------------------------------
pgxldocker_pgxl_1   /app/entrypoint.sh supervisor   Up      5432/tcp
pgxldocker_pgxl_2   /app/entrypoint.sh supervisor   Up      5432/tcp
pgxldocker_pgxl_3   /app/entrypoint.sh supervisor   Up      5432/tcp
pgxldocker_pgxl_4   /app/entrypoint.sh supervisor   Up      5432/tcp
```
Postgres-XL configuration

The script run.py use docker-py API to inspect the started containers and generate a configuration file for Postgres-XL.

The —ip argument displays the running containers:
```
$ python run.py --ip
```

The —conf command generates the configuration file:
```
$ python run.py --conf pom/pgxc_ctl
```
### 3. Starting Postgres-XL

To start easily Postgres-XL we use the tool pgxc_ctl that automates the launching of the services.

First, we connect to the coordinator container (coordMasterServers) using the generated ssh key.

```
$ ssh -i pom/.ssh/id_rsa pom@172.17.0.2
```
We start the pgxc_ctl tool:
```
$ pgxc_ctl
Installing pgxc_ctl_bash script as /pom/pgxc_ctl/pgxc_ctl_bash.
Installing pgxc_ctl_bash script as /pom/pgxc_ctl/pgxc_ctl_bash.
Reading configuration using /pom/pgxc_ctl/pgxc_ctl_bash --home /pom/pgxc_ctl --configuration /pom/pgxc_ctl/pgxc_ctl.conf
Finished to read configuration.
   ******** PGXC_CTL START ***************

Current directory: /pom/pgxc_ctl
PGXC
```

Postgres-XL initializations:
```
PGXC init all
```

### 4. Extend [PostGis](www.postgis.net)
To enable PostGIS (includes raster), you just need to connect a `coord` after initialize the PGXL.
```
$  psql -h 172.17.0.4 -U pom postgres

CREATE EXTENSION postgis;
```

### 5. Test Spatial SQL

```
-- Create table with spatial column
CREATE TABLE mytable (
  id SERIAL PRIMARY KEY,
  geom GEOMETRY(Point, 26910),
  name VARCHAR(128)
);

-- Add a spatial index
CREATE INDEX mytable_gix
  ON mytable
  USING GIST (geom);

-- Add a point
INSERT INTO mytable (geom) VALUES (
  ST_GeomFromText('POINT(0 0)', 26910)
);

-- Query for nearby points
SELECT id, name
FROM mytable
WHERE ST_DWithin(
  geom,
  ST_GeomFromText('POINT(0 0)', 26910),
  1000
);
```

### Authors

Matthieu Lagacherie , Yannick Drant and Xiaodong Wang

# MariaDB Galera Cluster
_Friedjof Noweck_

## Introduction
The MariaDB Galera Cluster is a synchronous multi-master cluster for InnoDB.
You can find three MariaDB Nodes in this Cluster.

## Documentation
### Network
Database `10.5.0.0/24`
- Node 0 -> `10.5.0.10` -> 00.mariadb.local
- Node 1 -> `10.5.0.11` -> 01.mariadb.local
- Node 2 -> `10.5.0.12` -> 02.mariadb.local
- Node 3 -> `10.5.0.13` -> 03.mariadb.local
### Data Directory
- Nodes:
  - Node 0 -> `data/mariadb/00`
  - Node 1 -> `data/mariadb/01`
  - Node 2 -> `data/mariadb/02`
  - Node 3 -> `data/mariadb/03`
  - conf.d -> `data/mariadb/conf.d` (MariaDB Configuration for Galera Cluster)
- ProxySQL:
  - Proxy 01 -> `data/proxysql/01`
  - Proxy 02 -> `data/proxysql/02`
  - proxysql.cnf -> `data/proxysql/proxysql.cnf` (ProxySQL Configuration)
### Setup
#### Copy the environment files
```bash
cp ./envs/00.env-template ./envs/00.env
cp ./envs/global.env-template ./envs/global.env
```
Now you can edit the environment files `00.env` and `global.env` to your needs.
- The `00.env` file contains the environment variables for the Bootstrap Node.
- The `global.env` file contains the environment variables for all Nodes.

**Node**: You might have to change the `MYSQL_ROOT_PASSWORD` in the `docker-compose.yml` and the `proxysql.cnf` files.
#### Bootstrap Node
The Bootstrap Node is the first Node in the Cluster. It is used to bootstrap the Cluster.
```bash
docker compose up -d 00.mariadb
```
Wait a few seconds for initialization.
#### Join Node 2 and 3
```bash
docker compose up -d 02.mariadb 03.mariadb
```
Wait a few minutes for the Cluster to be ready with the following messages:
```text
[Note] WSREP: Member 2.0 (02.mariadb.local) synced with group.
[Note] mysqld: ready for connections.
```
Show the Logs of the Node 2 and 3 to see the Cluster status.
```bash
docker logs -f 02.mariadb
```
```bash
docker logs -f 03.mariadb
```
#### Stop Node 0 and start Node 1
```bash
docker stop 00.mariadb
docker rm 00.mariadb
docker compose up -d 01.mariadb
```

### ProxySQL
#### Start ProxySQL
Run the SQLProxy on the Host.
```bash
docker compose up -d 01.proxysql 02.proxysql
```

## Reset the Repository
```bash
docker compose down
sudo rm -rf data/mariadb/01 data/mariadb/02 data/mariadb/03
```
```bash
mkdir data/mariadb/01 data/mariadb/02 data/mariadb/03
```
```bash
sudo touch data/mariadb/01/.gitkeep data/mariadb/02/.gitkeep data/mariadb/03/.gitkeep
```

## More useful commands
### Show running containers
```bash
docker compose ps
```
### Start Node 1, 2 and 3
```bash
docker compose up -d 01.mariadb 02.mariadb 03.mariadb
```
### Show the Cluster Status
```sql
select variable_name, variable_value from information_schema.global_status where variable_name in ("wsrep_cluster_size", "wsrep_local_state_comment", "wsrep_cluster_status", "wsrep_incoming_addresses")
```

## Sources
- [MariaDB Galera Cluster](https://mariadb.com/kb/en/mariadb/mariadb-galera-cluster/)
- https://severalnines.com/blog/running-mariadb-galera-cluster-without-container-orchestration-tools-part-one/
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
- Node 0 -> `data/00`
- Node 1 -> `data/01`
- Node 2 -> `data/02`
- Node 3 -> `data/03`
- conf.d -> `data/conf.d` (MariaDB Configuration for Galera Cluster)
### Setup
#### Copy the environment files
```bash
cp ./envs/00.env-template ./envs/00.env
cp ./envs/global.env-template ./envs/global.env
```
Now you can edit the environment files `00.env` and `global.env` to your needs.
- The `00.env` file contains the environment variables for the Bootstrap Node.
- The `global.env` file contains the environment variables for all Nodes.
#### Bootstrap Node
The Bootstrap Node is the first Node in the Cluster. It is used to bootstrap the Cluster.
```bash
docker compose up -d 00.mariadb
```
#### Join Node 2 and 3
```bash
docker compose up -d 02.mariadb 03.mariadb
```
Wait a few seconds for initialization.

Restart the two Nodes to join the Cluster.
```bash
docker compose restart 02.mariadb 03.mariadb
```
Wait a few minutes for the Cluster to be ready with the following messages:
```text
[Note] mysqld: ready for connections.
```
```text
[Note] WSREP: Member 2.0 (02.mariadb.local) synced with group.
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

## Reset the Repository
```bash
docker compose down
sudo rm -rf data/01 data/02 data/03
```
```bash
mkdir data/01 data/02 data/03
sudo touch data/01/.gitkeep data/02/.gitkeep data/03/.gitkeep
```
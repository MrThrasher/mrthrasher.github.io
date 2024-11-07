---
layout: post
title:  "MS SQL on Apple silicon"
date:   2022-12-05 13:00:26 +0200
category: development
---

## MS SQL docker container on ARM64

At this point there is no port of the origianl SQL Server for Linux image to ARM64 architecture. As a alternative there is a port of the Azure SQL Edge docker image.

[mcr.microsoft.com/azure-sql-edge](https://hub.docker.com/_/microsoft-azure-sql-edge)

This database is optimized for IoT and does not support every single feature of MS SQL Server, but can handle a normal database.

### Create the docker container

So let`s create the docker container for the Azure Sql Edge

```docker
docker run -e "ACCEPT_EULA=1" -e "MSSQL_SA_PASSWORD=MyPass@word" -e "MSSQL_PID=Developer" -e "MSSQL_USER=SA" -p 1433:1433 -d --name=sql mcr.microsoft.com/azure-sql-edge
```

After we created the container, we have to bring the database backup to the container. Therefore we have to create a backup folder

```docker
docker exec -it sql1 mkdir /var/opt/mssql/backup
```

Then we bring the database backup to this folder

```docker
docker cp yourdatabase.bak sql1:/var/opt/mssql/backup
```

Now we have to restore the backup to the database. This is possible with the Azure Data Studio. To do so, we have to connect to the docker container with Azure Data Studio and do the restore.

It could be, that we get an error message at this point, saying that we don`t have the permisson to restore the backup. In this case we have to change the permissons on the backup file.

```docker
docker exec -it -u root MicrosoftSQLServer "bash"
```

```docker
chown mssql /var/opt/mssql/backup/yourdatabase.bak
```

Then it should be possible to restore the bak file.
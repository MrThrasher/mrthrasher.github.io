---
layout: post
title:  "MS SQL on Apple silicon"
category: development
---

## Create the container for the Linux SqlServer

Run the SqlServer from the ubuntu sql server image from Microsoft. 

```docker
docker run -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=Tcpos@2020' --name 'sql1' -p 1401:1433 -v sql1data:/var/opt/mssql -d mcr.microsoft.com/mssql/server:2019-CU3-ubuntu-18.04
```

## Create a backup folder inside the container
```docker
docker exec -it sql1 mkdir /var/opt/mssql/backup
```

## Copy the bak file in the created backup folder
```docker
docker cp tcpos.bak sql1:/var/opt/mssql/backup
```

## Check the bak file for mdf (data) and ldf (log)
```docker
docker exec -it sql1 /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'Tcpos@2020' -Q 'RESTORE FILELISTONLY FROM DISK = "/var/opt/mssql/backup/tcpos.bak"' | tr -s ' ' | cut -d ' ' -f 1-2
```

## Provides this output
LogicalName PhysicalName
------------------------
tcpos       C:\Program
tcpos_log   C:\Program

(2 rows)

## Copy data and log to the needed location
```docker
docker exec -it sql1 /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'Tcpos@2020' -Q 'RESTORE DATABASE tcpos FROM DISK = "/var/opt/mssql/backup/tcpos.bak" WITH MOVE "tcpos" TO "/var/opt/mssql/data/tcpos.mdf", MOVE "tcpos_log" TO "/var/opt/mssql/data/tcpos.ldf"' 
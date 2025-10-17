## comando para ejecutar en linux
```docker
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=niko@123_docker!" \
-1422:1433 --name sqlserverBI \
-v sqlserver-volume:/var/opt/mssql \
-d docker pull mcr.microsoft.com/mssql/server:2022-latest
```
## comando para ejecutar en windows
```docker
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=niko@123_docker!" `
-1422:1433 --name sqlserverBI `
-v sqlserver-volume:/var/opt/mssql `
-d docker pull mcr.microsoft.com/mssql/server:2022-latest
```


Mipassword123!
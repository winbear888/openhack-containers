# Dockerfile mapping
0 - user-java
1 - tripviewer (webapp)
2 - userprofile (node)
3 - poi
4 - trip API

# Task 1a
From src/poi, run `docker build -t poi -f ..\..\dockerfiles\Dockerfile_3 .`

Log in to ACR
`az login`
`az acr login --name registryzjk6277`

Add login server to tag name if not done already
`docker tag userprofile registryzjk6277.azurecr.io/userprofile`

Push to ACR `docker push registryzjk6277.azurecr.io/<my-tag>`

# Task 1b
Download sql server image
`docker pull mcr.microsoft.com/mssql/server`

Run SQL server
`docker run -e 'ACCEPT_EULA=Y' mcr.microsoft.com/mssql/server`
`docker run -e 'ACCEPT_EULA=Y' -e "SA_PASSWORD=jamie123" -p 1433:1433 --name spl1 -h sql1 -d mcr.microsoft.com/mssql/server`

We need a dedicated docker network, DNS and some other things dont work on default network.
3 containers on network: SQL, data-load, poi.

`docker network create trip_insights`


Run SQL server on network
`docker run -e 'ACCEPT_EULA=Y' -e "SA_PASSWORD=jamie123" --network trip_insights -p 1433:1433 --name sql1 -h sql1 -d mcr.microsoft.com/mssql/server`

Verify network is associated to the container.
`docker inspect sql1`


Go into container to create table

`docker exec -it sql1 /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "jamie123"`
1> create database testDB
2> go


copy all data into table 
`docker run --network trip_insights -e SQLFQDN=<servername> -e SQLUSER=SA -e SQLPASS=jamie123 -e SQLDB=testDB openhack/data-load:v1`

Run POI with all environment variables. Refer to README in the poi folder.
`docker run -d -p 8080:80 --name poi --network trip_insights -e "SQL_PASSWORD=jamie123" -e "SQL_SERVER=sql1" -e "SQL_DBNAME=testDB" -e "ASPNETCORE_ENVIRONMENT=Local" -e "SQL_USER=sa" registryzjk6277.azurecr.io/poi`

Test API endpoints.

http://localhost:8080/api/poi/ to load from DB
http://localhost:8080/api/poi/healthcheck to check container

# Task 2
`kubectl create secret generic test1 --from-literal=username=jamie --from-literal=password=jamie123`

# Task 2 Links
https://docs.microsoft.com/en-us/azure/key-vault/general/key-vault-integrate-kubernetes
https://github.com/Azure/secrets-store-csi-driver-provider-azure
https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/
https://kubernetes.io/docs/concepts/configuration/secret/
https://kubernetes.io/docs/concepts/configuration/secret/#use-case-pods-with-prod-test-credentials
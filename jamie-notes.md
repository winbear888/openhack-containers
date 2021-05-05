# Dockerfile mapping
0 - user-java
1 - tripviewer (webapp)
2 - userprofile (node)
3 - poi
4 - trip API

# Task 1
From src/poi, run `docker build -t poi -f ..\..\dockerfiles\Dockerfile_3 .`
`az login`
`az acr login --name registryzjk6277`

Add namespace `docker tag userprofile registryzjk6277.azurecr.io/userprofile`
Push to ACR `docker push registryzjk6277.azurecr.io/userprofile`

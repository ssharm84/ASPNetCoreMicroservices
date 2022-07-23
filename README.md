# ASPNetCoreMicroservices
https://medium.com/aspnetrun/microservices-architecture-on-net-3b4865eea03f

image.png

1.  Create a Github Repository. Include, GitIgnore,Readme.md & License
2.  In VS, clone the repository.
3.  Create a sub-folder as src
4.  Add a blank solution in src...aspnetcore-microservices
5.  Add a folder in the solution named as services.Right click on services and add a new solution folder-Catalog
6.  Right click on Catalog and add a new ASP.Net Core Web API project and name it as Catalog.API. Make sure that it is created under Catalog folder.
7.  Right click on Catalog.API and select Properties->Debug and change the Profile from IIS Express to Catalog.API and Launch as Project. You will see App URL as http://localhost:5000
8.  Go to appsettings.json and you will see Catalog.API profile created.
9.  Run the first Microservice by selecting Catalog.API in the run button instead of IIS Express
10. Download Docker Desktop and go to hub.docker.com and search for mongo.......https://hub.docker.com/_/mongo
11. Right click on your solution and open Terminal and check if Docker is running...docker ps
12. docker login --username=ss84
13. docker pull mongo
14. docker run -d -p 27017:27017 --name shopping-mongo mongo......Here shopping-mongo database is created in port number 27017 from mongo image
15. Check if the image is running using...docker ps
16. Now to get into the mongo-shopping container....docker exec -it shopping-mongo /bin/bash
17. You will be in the container...mongo..............Command to get into your mongodb
18. show dbs.....to see the list of db
19. use CatalogDb.......to create a new Db and get into it
20. db.createCollection('Products').........cmd to create a new table Products
21. db.Products.insertMany([{ 'Name':'Asus Laptop','Category':'Computers', 'Summary':'Summary', 'Description':'Description', 'ImageFile':'ImageFile', 'Price':54.93 }, { 'Name':'HP Laptop','Category':'Computers', 'Summary':'Summary', 'Description':'Description', 'ImageFile':'ImageFile', 'Price':88.93 } ])
    or
    db.Products.insertMany(
			[
			    {
			        "Name": "Asus Laptop",
			        "Category": "Computers",
			        "Summary": "Summary",
			        "Description": "Description",
			        "ImageFile": "ImageFile",
			        "Price": 54.93
			    },
			    {
			        "Name": "HP Laptop",
			        "Category": "Computers",
			        "Summary": "Summary",
			        "Description": "Description",
			        "ImageFile": "ImageFile",
			        "Price": 88.93d
			    }
			])
22. show collections
23. db.Products.find().pretty().........to see the json data in table
24. db.Products.Remove()...........cmd to delete table data
25. Layers in a Microservice:
    a.  API/Application Layer-Entry point into service.Exposes endpoints & enforces validation. No Business logic
    b.  Domain Layer - Heart of the software. Contains business rules & logic. Business operations are implemented here.
    c.  Infrastructure Layer - Primary responsibility is persistence of business state.
26. So, our API Architecture will be - API->Business Object->Repository->Db
27. In Repository Pattern Application communicates with DAL through the Interfaces in Repository layer. Here Business logic will indirectly interact with DAL through   Repository. As a result there is an abstraction of DAL using Repository Layer.
28. Now get into solution and install MongoDB driver from Nuget either using Package Manager Console or Nuget...Go to nuget.org for commands
29. Now add Entities folder and inside it add Product class and entities here.
30. Next step is to create a Data Layer to connect Catalog.API with MongoDB.
31. In appsettings.json, paste this connection string at the top:
"DatabaseSettings": {
    "ConnectionString": "mongodb://localhost:27017",
    "DatabaseName": "ProductDb",
    "CollectionName": "Products"
  },
32. Create a Data Layer by creating Data folder and then create an Interface ICatalogContext and its definition class CatalogContext.cs class. Also create a CatalogContextSeed.cs class
33. Create a Business Layer in the form of Repositories folder and inject ICatalogContext into ProductRepository class which is implementing IProductRepository
34. Develop Presentation Layer/API Layer by Creating CatalogController.
35. Now the API is ready so we need to update the package by running the following command: Update-Package -ProjectName Catalog.API

36. Containerize Catalog Microservices with MongoDb
    a.Right click on Catalog.API and select Add->Container Orchestrator Support->Docker Compose->Linux
    b.It will result in creation of dockerfile and docker-compose
    c.Update dockerfile, docker-compose.yml and docker-compose.override.yml for catalogapi,catalogdb
    1.  dockerfile:
    #See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
WORKDIR /src
COPY ["services/Catalog/Catalog.API/Catalog.API.csproj", "services/Catalog/Catalog.API/"]
RUN dotnet restore "services/Catalog/Catalog.API/Catalog.API.csproj"
COPY . .
WORKDIR "/src/services/Catalog/Catalog.API"
RUN dotnet build "Catalog.API.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "Catalog.API.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Catalog.API.dll"]

    2.  docker-compose.override.yml

version: '3.4'

services:
  catalogdb:
    container_name: catalogdb
    restart: always
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

  catalog.api:
    container_name: catalog.api
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - "DatabaseSettings:ConnectionString=mongodb://catalogdb:27017"
    depends_on:
      - catalogdb
    ports:
      - "8000:80"

      3.    docker-compose.yml

version: '3.4'

services:
  catalogdb:
    image: mongo

  catalog.api:
    image: ${DOCKER_REGISTRY-}catalogapi
    build:
      context: .
      dockerfile: services/Catalog/Catalog.API/Dockerfile

volumes:
  mongo_data:

    d.  Right click on docker-compose project and Open in terminal
    e.  docker ps.....Check for the running container and if a container is running on 27017 port , then type: docker stop {containerid first 4 letters}
    f.  docker ps..to verify if no other container is running on that port
    g.  Remove the container...docker rm cced....before removing..docker ps -a..to check for active conatiner and then docker stop cced and lastly docker rm cced
    h.  docker-compose -f docker-compose.yml -f docker-compose.override.yml up --build
    i.  After this check if both the containers are activated for mongo and API
    j.  Go to http://localhost:8000/swagger/index.html and check if the docker image of api is running fine
    k.  Setting for docker....Open Tools->Options->Search Docker->Docker Compose and set options there as False, Never, True, False
    l.  To stop and remove all containerized images - docker-compose -f docker-compose.yml -f docker-compose.override.yml up

Service 2: Basket.API
1.  Right click services and add a new folder named Basket
2.  Right click Basket and add a new ASP.Net Core Web API project and name it as Basket.API
3.  Right click on Basket.API and select Properties->Debug and change the Profile from IIS Express to Basket.API and Launch as Project.Also, change port number to 5001
4.  Right click on project and Set as Startup Project.
5.  Now we need to have our data source as Redis which is:
    a.  Open-source NoSQL Database
    b.  Remote dictionary server, key-value pairs, Data Structure server, Extremely fast
6.  Right click on docker-compose and open terminal and type the following command to pull redis image from docker hub: docker pull redis
7.  Before Point 6, run this command - docker login --username=ss84
8.  docker images to check for redis and then: docker run -d -p 6379:6379 --name aspnetrun-redis redis
9.  To check the logs of any image: docker logs -f aspnetrun-redis
10. docker exec -it aspnetrun-redis /bin/bash....where it is interactive terminal
11. After that, we are able to run redis commands. 
    a.  redis-cli
    b.  ping - PONG
    c.  set key value
    d.  get key...."value"
    e.  set name steve
    f.  get name...."steve"
12. Inside Business logic or Service Layer - It processes the data taken from DAL into the Project. So, the data coming from the user will go to the BLL and after      that data is transferred to DAL.
13. Project Folder Structure:
    a.  Entities Folder - storing the Redis Entities
    b.  Data Folder-which will be the data Context class
    c.  Repositories - where we will be applying Business Logic
    d.  Controller - Exposing APIs to external systems
14.       





# ASPNetCoreMicroservices

1.Create a Github Repository. Include, GitIgnore,Readme.md & License
2.In VS, clone the repository.
3.Create a sub-folder as src
4.Create a blank solution in src...aspnetcore-microservices
5.Add a solution folder in the solution-services.Right click on services and add a new solution folder-Catalog
6.Right click on Catalog and add a new ASP.Net Core Web API project and name it as Catalog.API. Make sure that it is created under Catalog folder.
7.Right click on Catalog.API and select Properties->Debug and change the Profile from IIS Express to Catalog.API and Launch as Project. You will see App URL as http://localhost:5000
8.Go to appsettings.json and you will see Catalog.API profile created.
9.Run the first Microservice by selecting Catalog.API in the run button instead of IIS Express
10.Download Docker Desktop and go to hub.docker.com and search for mongo.......https://hub.docker.com/_/mongo
11.Right click on your solution and open Terminal and check if Docker is running...docker ps
12.docker login --username=ss84
13.docker pull mongo
14.docker run -d -p 27017:27017 --name shopping-mongo mongo......Here shopping-mongo database is created in port number 27017 from mongo image
15.Check if the image is running using...docker ps
16.Now to get into the mongo-shopping container....docker exec -it shopping-mongo /bin/bash
17.You will be in the container...mongo..............Command to get into your mongodb
18.show dbs.....to see the list of db
19.use CatalogDb.......to create a new Db and get into it
20.db.createCollection('Products').........cmd to create a new table Products
21.db.Products.insertMany([{ 'Name':'Asus Laptop','Category':'Computers', 'Summary':'Summary', 'Description':'Description', 'ImageFile':'ImageFile', 'Price':54.93 }, { 'Name':'HP Laptop','Category':'Computers', 'Summary':'Summary', 'Description':'Description', 'ImageFile':'ImageFile', 'Price':88.93 } ])
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
22.show collections
23.db.Products.find().pretty().........to see the json data in table
24.db.Products.Remove()...........cmd to delete table data
25.Layers in a Microservice:
    a.  API/Application Layer-Entry point into service.Exposes endpoints & enforces validation. No Business logic
    b.  Domain Layer - Heart of the software. Contains business rules & logic. Business operations are implemented here.
    c.  Infrastructure Layer - Primary responsibility is persistence of business state.
26. So, our API Architecture will be - API->Business Object->Repository->Db
27.In Repository Pattern Application communicates with DAL through the Interfaces in Repository layer. Here Business logic will indirectly interact with DAL through Repository. As a result there is an abstraction of DAL using Repository Layer.
28.Now get into solution and install MongoDB driver from Nuget either using Package Manager Console or Nuget...Go to nuget.org for commands
29.Now add Entities folder and inside it add Product class and entities here.
30.Next step is to create a Data Layer to connect Catalog.API with MongoDB.
31.In appsettings.json, paste this connection string at the top:
"DatabaseSettings": {
    "ConnectionString": "mongodb://localhost:27017",
    "DatabaseName": "ProductDb",
    "CollectionName": "Products"
  },
32.  


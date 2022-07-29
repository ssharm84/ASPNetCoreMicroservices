# ASPNetCoreMicroservices
https://medium.com/aspnetrun/microservices-architecture-on-net-3b4865eea03f

![image](https://user-images.githubusercontent.com/67266176/180625649-f28d8a9d-3934-4b5f-aaa0-5a10443939b3.png)


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
12. TLS Handshake Timeout issue...docker login --username=ss84 --password=India@123
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
33. Create a Business Layer in the form of Repositories folder and inject ICatalogContext into ProductRepository class which is implementing IProductRepository.
    In Startup.cs, add the Interfaces & classes of Data & Repository layers
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
14. Create Entities folder and add a new class called ShoppingCart.cs which will have username and ShoppingListItem as the properties. 
15. For ShoppingListItem,create a new class and include all the properties of the Shopping Items
16. In ShoppingCart, add another method for Total Price.
17. In Package Manager Console-Select Basket.API....cmd-Install-Package Microsoft.Extensions.Caching.StackExchangeRedis
18. Update-Package -ProjectName Basket.API
19. In appsettings.json, add the connection string:
    "CacheSettings": {
    "ConnectionString": "localhost:6379"
  }
20. In Startup.cs, add the following code to ensure interaction with Redis:
    services.AddStackExchangeRedisCache(options =>
            {
                options.Configuration = Configuration.GetValue<string>("CacheSettings.ConnectionString");
            });
10. Create Repositories folder and add an Interface and its definition class.
11. Now create BasketController.cs file and inject IBasketRepository. Also add IBasketRepository in Startup.cs
12. Now check whether Redis image is running, Right click on Solution and type: docker ps

Service 3: Discount.API
1.	Create a folder Discount under Services
2.	Add a new ASP.Net Core Web API project with the name Discount.API. Make sure that the location is correct.
3.	.Net 5.0, Authentication Type: None, Uncheck Configure for Https
4.	Right click on Discount.API->Properties->Debug and change Profile to Discount.API and Launch: Project and Application URL: http://localhost:5002
5.	Set Discount.API as startup page and run it.
6.	Click on docker-compose.yml and add the following code under basketdb in the services section:
	discountdb:
	  image: postgres
7.	Get into docker-compose.override.yml and add a section for dockerdb....This override file is for adding the configurations of all the databases and services
8.	Come back to docker-compose.yml and in the volumes section add:
	volumes:
	  mongo_data:
	  portainer_data:
	  postgres_data:
9.	Now we need to add pgadmin4 image which is the management tool of postgresql into our application.
10.	In docker-compose.yml, add a section below discountdb:
	pgadmin:
	  image: dpage/pgadmin4
11.	In volumes add- pgadmin_data:	
12.	In docker-compose.override.yml, add the environment variable & configuration for pgadmin
13.	Right click docker-compose and Open in terminal
14.	docker-compose up command: docker-compose -f docker-compose.yml -f docker-compose.override.yml up -d
15.	In this way the docker images of postgresql and pgadmin will be installed
16. 	docker ps........will show the running containers on the local machine
17.	If there is some issue or any of the image is not installed, get into portainer.io using localhost:9000 and credentials->admin|admin1234	
18.	In Containers section you will see all the images that are running and can check if any of the service is not running by clicking the Container and check 	the Logs
19.	Open localhost:5050 for verifying if pgadmin is working fine....Look for the credentials in docker-compose.override.yml file
20.	Create a new Server called DiscountServer
21.	Inside Connection->Host name = discountdb and keep every other settings same. Username=admin,pwd=admin1234...verify in in docker-compose.override.yml file
22.	Inside database you will now see DiscountDb database.Inside Schemas, you can see Tables.
23.	Now we need to create Coupon table..Click Tools->Query Tool
	Create Table Coupon(
		ID SERIAL PRIMARY KEY NOT NULL,
		ProductName VARCHAR(24) NOT NULL,
		Description TEXT,
		Amount INT
	);
24.	To insert Data...Right click coupon, View/Edit Data..to see the data in the table.
25. 	Insert queries:
	Insert into Coupon(ProductName,Description,Amount) Values ('IPhone X','IPhone Discount',150);
	Insert into Coupon(ProductName,Description,Amount) Values ('Samsung 10','Samsung Discount',100);
26.	Right click on Discount.API and add a new folder named Entities and add a class-Coupon.cs & add the following properties-Id,ProductName,Description,Amount
27.	Open package manager console and type: Install-Package Npgsql & Install-Package Dapper
28.	Update-Package -ProjectName Discount.API
29.	Create Repositories folder->IDiscountRepository.cs and then DiscountRepository.cs and write CRUD functions in it.
30.	Add connection string in appsettings.json
31.	Now create DiscountController class.
32.	Add the Repositories in Startup.cs
33.	Containerize Discount Microservices with PostgresDb
    	a.	Right click on Discount.API and select Add->Container Orchestrator Support->Docker Compose->Linux
    	b.	It will result in creation of dockerfile and will update docker-compose with discount.api
    	c.	Update dockerfile, docker-compose.yml and docker-compose.override.yml for Discount.API,discountdb
	d.	In docker-compose.override.yml, update discount.api with postgres server connection
34.	Now for migrating PostgreSQL database when Discount Microservices starts, go to Program.cs and in Main(), add the below code
	a.	var host = CreateHostBuilder(args).Build();
		host.MigrateDatabase<Program>();
		host.Run();
	b.	Remove the other code in Main()
35.	Now we need to create an extension method named MigrateDatabase. For that create a folder called Extensions and add a class file-HostExtension.cs
36.	Over here we will be performing Retry operation but an alternate for this is to use Polly
36.	Add a new folder-Extensions and add a class HostExtensions.cs and Install-Package Polly
37. In Main() of Program.cs, add the folloving code:
  var host = CreateHostBuilder(args).Build();
            host.MigrateDatabase<Program>();
            host.Run();

Discount.gRPC
1.  Google Remote Procedure Call(Grpc) is a synchronous backend microservice-to-microservice communication and is implemented where performance is critical
2.  Right click on Discount folder->Add a new project->Search with gRPC and create ASP.Net Core gRPC Service, Project Name-Discount.Grpc
3.  In the folder structure, Protos folder has the gRPC service
4.  We are going to create a new proto file for exposing discount to Basket Microservices.
5.  Services folder performs the gRPC connection. This folder is equivalent to Controller folder. In greet.proto, it is the definition of. GreetService provides abstraction of methods to proto files.
6.  gRPC uses Http2 protocol for exposing services.
7.  proto files use Protobuf compiler to convert code into C#
8.  Right click the project to generate C# class from proto file. Click Show All files. Go to obj->Debug->net5.0->Protos->Greet.cs
9.  Right click on project->Debug->Profile=Discount.Grpc, launch=Project and change the port number to 5003 and remove https
10. Set as startup project and run the application. It will not open any browser since it is running on Http2 protocol
11. Install Dapper & NpgSql packages from Nuget or copy the references from Discount.API project
12. In Package Manager Console, Update-Package -ProjectName Discount.Grpc
13. Copy Entities folder from Discount.API and paste it in Discount.Grpc folder
14. Copy Repositories folder & paste in Grpc project
15. Copy Extensions folder & paste in Grpc project
16. Copy appsettings connection string for connecting to database
17. Also, in Startup.cs, add IDiscountRepository.cs & DiscountRepository.cs 
18. In Main() of Program.cs, add the folloving code:
  var host = CreateHostBuilder(args).Build();
            host.MigrateDatabase<Program>();
            host.Run();
19. So here we are concentrating on how to create gRPC service in order to consume from Basket.API
20. Let's test now. So, right click docker-console->Open in Terminal->docker-compose -f docker-compose.yml -f docker-compose.override.yml up -d
21. Right click Discount.Grpc & set as Startup project.
22. Now in order to expose the endpoints, we need to create Services and protos. These Services are representing in API as Controller
23. First, delete greet.proto and GreeterService.cs file.
24. In Startup.cs, comment out the endpoint.
25. Right click Protos->Add New Item, Protocol Buffer File -> discount.proto and click Add button:
	syntax = "proto3";.....This one is saying that we are using protobuf version as proto3 
	option csharp_namespace = "Discount.Grpc.Protos";
	
	service DiscountProtoService{..........................This is a Grpc Service
		rpc GetDiscount (GetDiscountRequest) returns (CouponModel);
		rpc CreateDiscount (CreateDiscountRequest) returns (CouponModel);
		rpc UpdateDiscount (UpdateDiscountRequest) returns (CouponModel);
		rpc DeleteDiscount (DeleteDiscountRequest) returns (DeleteDiscountResponse);
	}
	
	message GetDiscountRequest {
		string productName = 1;
	}
	
	message CouponModel {
		int32 id = 1;
		string productName = 2;
		string description = 3;
		int32 amount = 4;
	}
	
	message CreateDiscountRequest {
		CouponModel coupon = 1;
	}
	
	message UpdateDiscountRequest{
		CouponModel coupon = 1;
	}

  message DeleteDiscountRequest {
    string productName = 1;
  }
	
	message DeleteDiscountResponse{
		bool success = 1; 
	}

26. Now we are going to generate proto service from discount.proto file
27. Right click on discount.proto and select Properties, change Build Action = Protobuf compiler, gRPC Sub classes=Server only since we are going to create gRPC server for exposing Discount Services      
28. Add DiscountService.cs class under Services folder and inject IDiscountRepository and ILogger. 
29. Press enter and click on the down arrow dropdown icon on the left and select Generate Constructor
30. Now type override and press space and you will see all the methods that are in the proto file.
31. GetDiscount method where remove the default code and write the code.
32. In Startup.cs, uncomment the endpoints and write DiscountService.
33. Install-Package AutoMapper.Extensions.Microsoft.DependencyInjection
  a.  AutoMapper - It is an object to object mapper i.e. it maps an object of one type to another type.  
34. Create a new folder called Mapper and create a class DiscountProfile.cs to create Mapping for Coupon & CouponModel
  a.  This class will implement Profile class from AutoMapper and will use CreateMap method to map Coupon from Entities with CouponModel from proto class
  public class DiscountProfile:Profile
    {
        public DiscountProfile()
        {
            CreateMap<Coupon, CouponModel>().ReverseMap();
        }
    }
35. Now go back to DiscountService and inject IMapper and write all the endpoints.

Section 6: Consuming Discount Grpc Service from Basket Microservice when adding Cart Item into Shopping Cart to calculate Final Price
1.  Open Basket.API project, open BasketController->UpdateBasket method and there we need to communicate with Discount.Grpc and calculate latest prices into Shopping cart.
2.  So, now Basket.API will be the client of Discount.Grpc
3.  Right click Basket.API>Add->Connected Service->In Service Reference click Add a Service Reference->Select gRPC->Next->Browse Discount.proto file->Type of class to be generated = Client since we are going to consume this gRPC method->Finish
4.  Double click Basket.API project & you will see a new Item Group = Protobuf
5.  You will also find Protos folder with discount.proto file. Right click on it & you will see Client only as gRPC sub classes
6.  Now, build the project. Next step is to consume gRPC service from Basket.API microservices when adding cart-item into the Shopping Cart to calculate final price of the product into the Cart item.
7.  Right click Basket.API->Add a new folder:GrpcService and add a new class DiscountGrpcService.cs & inject DiscountProtoService.DiscountProtoServiceClient object.
8.  Since it is a client application so we are injecting DiscountProtoService and not inheriting.
9.  Create a constructor. In the GetDiscount method we make Grpc call with discountRequest object.
10. Now we need to consume this service in our Basket.API Controller. So inject this service in the controller, add it in the constructor.
11. Go to UpdateBasket method in Controller and for getting the discount of each item, write a foreach loop and perform call to Grpc GetDiscount method with the productName
12. After this we will deduct the final price with the coupon Amount.
13. Now in order to register DiscountGrpc client and DiscountGrpc services into Startup class:
    // Grpc Configuration
            services.AddGrpcClient<DiscountProtoService.DiscountProtoServiceClient>
                        (o => o.Address = new Uri(Configuration["GrpcSettings:DiscountUrl"]));
            services.AddScoped<DiscountGrpcService>();
14. In appsettings.json, add GrpcSettings configuration:
  "GrpcSettings": {
    "DiscountUrl":  "http://localhost:5003"
  }
15.  Port number can be verified by right clicking on Discount.Grpc and select Properties
16. We are putting in the localhost ports. However once we containerize them then we will override the ports in docker-compose.override.yml configuration.
17. Right click on Solution->Properties->Startup Project->Select Multiple Startup Projects and select action of Basket.API & DiscountGrpc as Start and Apply.
18. Run the application and put the breakpoint in UpdateBasket Task & pass the following json input in http://localhost:5001/api/v1/Basket:
    {
    "UserName": "Steve",
    "Items": [
    {
    "Quantity": 2,
    "Color": "Red",
    "Price": 500,
    "ProductId": "60210c2a1556459e153f0555",
    "ProductName": "Samsung 10"
    },
    {
    "Quantity": 1,
    "Color": "Blue",
    "Price": 400,
    "ProductId": "60220c3a1556459e153f0555",
    "ProductName": "IPhone X"
    }
]
}

19. You will see that in the output body the price is adjusted with discount.
20. Now we need to create containerized image of Discount.Grpc
21. Right click on project->Add->Docker Orchestrator Support
22. In docker-compose.override, add section for discount.grpc
23. Also, remember that we added grpc connection string in basket.api appsettings.json. So, add in this file too.
24. Now we need to rebuild the docker images, so we will go with --build, otherwise for updating, we go with -d
25. Right click docker-compose, Open in Terminal
26. docker-compose -f docker-compose.yml -f docker-compose.override.yml up --build.....This will build all the docker images again
27. Get into postgresdb by logging into localhost:5050..Add Server->Server=DiscountServer, Host name = discountdb
28. Portainer..localhost:9000....admin|admin1234567

Ordering API:
1. 


	



